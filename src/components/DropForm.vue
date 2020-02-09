<template>
  <div class="container">
  <form v-on:drop="handleDrop" class="drag-area">
    <div class="disabled-state">
      <div class="lds-ellipsis"><div></div><div></div><div></div><div></div></div>
    </div>
    <div class="enabled-state">
      <upload-icon size="1.5x" class="custom-class"></upload-icon>
      <label for="uploader">Drop your .png or .jpg file here!</label>
      <input type="file" accept="image/*" name="uploaded">
    </div>
  </form>

  <div class="results">

  </div>
</div>
</template>

<script>
/*eslint-disable */
import { UploadIcon } from 'vue-feather-icons'
import { saveAs } from 'file-saver';
import ImageTools from '../scripts/imagetools.min.js'
const JSZip = require("jszip");

let zip = new JSZip();
let DOM;

export default {
  name: 'DropForm',
  components: {
    UploadIcon
  },
  methods: {
    handleDrop: function(e) {
      const dt = e.dataTransfer
      const files = dt.files
      this.handleFiles(files)

      DOM.dragForm.removeEventListener('drop', this.handleDrop, false);
      DOM.dragForm.classList.add('disabled');
    },
    handleFiles: function(files) {
      ([...files]).forEach(this.uploadFile)
    },
    uploadFile: function(file) {
      const emoteSizes = [28,56,112];
      let modifiedImages = [];
      let imagesProcessed = 0;

      const ev = this;

      emoteSizes.forEach(function(size, index, array) {
        ImageTools.resize(file, {
          width: size,
          height: size
        }, function(blob, didItResize) {
          imagesProcessed++;
          modifiedImages.push(blob)

          if(imagesProcessed === array.length) {
            //Sort images biggest to smallest
            const sorted = modifiedImages.sort(function(a,b) {return parseFloat(a.size) - parseFloat(b.size);})
            ev.displayEmotes(sorted.reverse());
          }
        })
      })
    },
    displayEmotes: function(modifiedImages, size) {
      const ev = this;
      let imagesProcessed = 0;
      modifiedImages.forEach(function(imageBlob, index, array){
        const parent = document.createElement('div');
        const imageWrap = document.createElement('div');
        const size = document.createElement('p');
        const canvas = document.createElement('canvas');
        const context = canvas.getContext('2d');
        const baseImage = new Image();

        parent.classList.add('result-wrap');
        imageWrap.classList.add('result');

        baseImage.src = window.URL.createObjectURL(imageBlob);
        baseImage.onload = function(){
          canvas.height = baseImage.height;
          canvas.width = baseImage.width;
          size.textContent = `${baseImage.width}px`;
          context.drawImage(baseImage, 0,0, canvas.width, canvas.height);

          let blobToPng = ev.convertToPng(canvas)
          blobToPng.name = `${baseImage.width}`;

          const fileName = `${blobToPng.name}.png`;
          zip.file(fileName, imageBlob);
          imagesProcessed++;
          if(imagesProcessed === array.length) {
            zip.generateAsync({type:"blob"})
            .then(function(content) {
              saveAs(content, Date.now());
            });
          }
        }

        imageWrap.appendChild(canvas);
        parent.appendChild(imageWrap);
        parent.appendChild(size);
        DOM.results.appendChild(parent);
      })



      DOM.dragForm.classList.remove('disabled');
    },
    convertToPng: function(canvas) {
      const image = new Image();
      image.src = canvas.toDataURL("image/png");
      return image;
    }
  },
  mounted() {
    DOM = {
      dragForm: document.querySelector('.drag-area'),
      dragInput: document.querySelector('input[type="file"]'),
      results: document.querySelector('.results')
    }

    //Prevent defaults
    ;['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
      DOM.dragForm.addEventListener(eventName, preventDefaults, false);
    })

    function preventDefaults(e) {
      e.preventDefault()
      e.stopPropagation()
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
