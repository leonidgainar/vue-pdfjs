<template>
<div>
  <section v-if="!isFilled" class="actions">
    <button @click="sendData"> Send to server </button>
    <button @click="getData"> Get from server </button>
  </section>
  <div id="pageContainer">
    <div id="viewer" class="pdfViewer"></div>
  </div>
</div>
</template>

<script>
import pdfjsLib from "pdfjs-dist/build/pdf";
import { PDFViewer } from "pdfjs-dist/web/pdf_viewer";
import { PDFDocument, PDFTextField, PDFCheckBox, PDFDropdown, PDFRadioGroup } from 'pdf-lib';
import "pdfjs-dist/web/pdf_viewer.css";

pdfjsLib.GlobalWorkerOptions.workerSrc =
  "https://cdn.jsdelivr.net/npm/pdfjs-dist@2.0.943/build/pdf.worker.min.js";

const API_URL = process.env.VUE_APP_API_URL;

export default {
  name: "PdfViewer",
  props: {
    file: {
      type: String,
      required: true
    },
  },
  data() {
    return {
      isFilled: false,
    }
  },
  mounted() {
    this.setPdfToViewer(this.file);
  },
  methods: {
    async setPdfToViewer(url) {
      const container = document.getElementById("pageContainer");
      const pdfViewer = new PDFViewer({
        container: container,
        renderInteractiveForms: true,
        textLayerMode: 1,
      });
      const loadingTask = pdfjsLib.getDocument(url);
      loadingTask.then((pdf) => {
        pdfViewer.setDocument(pdf);
      });
    },
    async getDataFromPdf(url) {
      const pdfDoc = await PDFDocument.load(url);
      const form = pdfDoc.getForm();
      const fields = form.getFields();
      const pdfFields = [];
      fields.map((field) => {
          if (field instanceof PDFTextField) {
            pdfFields.push({
              name: field.getName(), 
              value: field.getText(), 
              id: field.ref.tag.replace(/\s+0\s/g, '')
            });
            return;
          }
          if (field instanceof PDFCheckBox) {
            pdfFields.push({
              name: field.getName(), 
              value: field.isChecked(), 
              id: field.ref.tag.replace(/\s+0\s/g, '')
            });
            return;
          }
          if (field instanceof PDFDropdown) {
            pdfFields.push({
              name: field.getName(), 
              value: field.getSelected(), 
              options: field.getOptions(),
              id: field.ref.tag.replace(/\s+0\s/g, '')
            });
            return;
          }
          if (field instanceof PDFRadioGroup) {
            const options = field.getOptions();
            const selected = field.getSelected();
            const fieldOptions = options.map((option) => {
              return {
                name: option, 
                value: option === selected,
              }
            });
            pdfFields.push({
              name: field.getName(), 
              options: fieldOptions,
              id: field.ref.tag.replace(/\s+0\s/g, '')
            });
            return;
          }
      });
      return pdfFields;
    },
    getDataFromHtml() {
      const htmlFields = [];
      const el = document.getElementsByClassName('annotationLayer')[0];
      for (let i = 0; i < el.children.length; i++) {
        switch(el.children[i].firstChild.type){
          case 'text': 
          case 'select-one': {
            htmlFields.push({
              value: el.children[i].firstChild.value.replace(/[^A-Za-z0-9\s!?]/g,''),
              id: el.children[i].getAttribute('data-annotation-id')
            });
            break;
          }
          case 'checkbox':
          case 'radio': {
            htmlFields.push({
              value: el.children[i].firstChild.checked,
              id: el.children[i].getAttribute('data-annotation-id')
            });
            break;
          }
        }
      }
      return htmlFields;
    },
    combineData(htmlData, pdfData) {
      const combinedData = [];
      htmlData.forEach((htmlItem) => {
        pdfData.forEach((pdfItem) => {
          if (htmlItem.id === pdfItem.id){
            combinedData.push({
              name: pdfItem.name,
              value: htmlItem.value
            });
          } else if (htmlItem.name === pdfItem.name) {
            if (htmlItem.value) {
              combinedData.push({
                name: pdfItem.name,
                value: pdfItem.options.find(option => option.value).name
              });
            }
          }
        });
      });
      return combinedData;
    },
    async fillPdfWithData(data) {
      const pdfDoc = await PDFDocument.load(this.file);
      const form = pdfDoc.getForm();
      const fields = form.getFields();
      
      fields.map((field) => {
        data.forEach((item) => {
          if (field.getName() === item.name) {
            if (field instanceof PDFTextField){
              field.setText(item.value);
            }
            if (field instanceof PDFCheckBox){
              item.value ? field.check() : field.uncheck();
            }
            if (field instanceof PDFRadioGroup || field instanceof PDFDropdown){
              field.select(item.value);
            }
          }
        });
      });
      this.isFilled = true;
      form.flatten();
      const pdfBytes = await pdfDoc.save();
      const blob = new Blob([pdfBytes], { type: "application/pdf" });
      const url = URL.createObjectURL(blob);
      this.setPdfToViewer(url);
      this.downloadFile(url, 'sample-pdf-form-filled.pdf');
    },
    downloadFile(path, filename){
      const anchor = document.createElement('a');
      anchor.href = path;
      anchor.download = filename;
      document.body.appendChild(anchor);
      anchor.click();
      document.body.removeChild(anchor);
    },
    async sendData() {
      const htmlData = this.getDataFromHtml();
      const pdfData = await this.getDataFromPdf(this.file);
      const combinedData = this.combineData(htmlData, pdfData);

      const formData = new FormData();
      formData.append("data", JSON.stringify(combinedData));
      fetch(`${API_URL}send_data`, {
        method: 'POST',
        body: formData,
      }).then((res) => { 
          if(res.status === 200) {
            this.$emit('data-sent');
          }
       }).catch((err) => ("Error occured on sending data", err));
    },
    getData() {
      fetch(`${API_URL}get_data`, {
        method: 'GET',
      }).then((res) => { 
        if(res.status === 200) {
          res.json().then((data) => {
            this.fillPdfWithData(data);
          });
        }
      }).catch((err) => ("Error occured on getting data", err));
    },
  },
};
</script>

<style scoped>
.actions > button {
  width: 200px;
  padding: 16px;
  margin: 8px;
  background-color: #3f51b5;
  font-size: 16px;
  color: #ffffff;
  border-radius: 4px;
  cursor: pointer;
}

.actions > button:hover {
  opacity: 0.8;
}
</style>
