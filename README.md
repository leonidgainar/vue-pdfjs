# vue-pdfjs - a project based on pdfjs-dist and pdf-lib

## How to use?
- Upload a PDF file that contains fillable fields (a sample can be find in the **src/assets** folder)
- Fill the fields from PDF with data
- Click on the **Send to server** button
- Upload the PDF file again
- Click on the **Get from server** button
- The PDF will be filled with data and downloaded
- Refresh browser window

### Live demo [here](https://vue-pdfjs.netlify.app/)
### Server repository: [vue-pdfjs-server](https://github.com/leonidgainar/vue-pdfjs-server)

## Project setup
```
yarn install
```

## Create .env.local file in root directory of the project and add the following content
```
VUE_APP_API_URL=http://localhost:5000/
```
## Clone [vue-pdfjs-server](https://github.com/leonidgainar/vue-pdfjs-server) repository and start the server
```
$ git clone git@github.com:leonidgainar/vue-pdfjs-server.git
$ cd vue-pdfjs-server
$ npm start
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```

