<template>
  <div id="app">
    <input v-show="!file" ref="inputFile" type="file" accept=".pdf" @change="onFileChange">
    <PdfViewer v-if="file" :file="file" :fileName="fileName" @data-sent="onDataSent"/>
  </div>
</template>

<script>
import PdfViewer from "./components/PdfViewer";

export default {
  name: "App",
  components: {
    PdfViewer
  },
  data() {
    return {
      file: null,
      fileName: ''
    }
  },
  methods: {
    onFileChange(e) {
      this.removePdf();
      const files = e.target.files || e.dataTransfer.files;
      if (!files.length)
        return;
      this.createPdf(files[0]);
    },
    createPdf(file) {
      const reader = new FileReader();
      const vm = this;
      
      reader.onload = (e) => {
        vm.file = e.target.result;
      };
      reader.readAsDataURL(file);
      this.fileName = file.name;
    },
    removePdf() {
      this.file = null;
      this.fileName = '';
    },
    onDataSent() {
      this.removePdf();
      this.$refs.inputFile.value = null;
    },
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 50px;
}
</style>
