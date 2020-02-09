<template>
  <div class="container">
  <form v-on:drop="handleDrop" v-bind:class="{active: isActive}" class="drag-area shadow">
    <div class="disabled-state">
      <div class="lds-ellipsis"><div></div><div></div><div></div><div></div></div>
    </div>
    <div class="enabled-state">
      <upload-icon size="1.5x" class="custom-class"></upload-icon>
      <label for="uploader">Drop your .png or .jpg file here!</label>
      <input type="file" accept="image/*" name="uploaded">
    </div>
  </form>

  <div class="results"></div>
  <div class="download"></div>

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
  data() {
    return {
      isActive: false
    }
  },
  methods: {
    handleDrop: function(e) {
      //Check if file handling is in process
      if(this.isActive === false) {
        //Toggle active class
        this.isActive = !this.isActive;
        const files = e.dataTransfer.files

        zip.remove();
        DOM.results.innerHTML = '';
        DOM.download.innerHTML = '';
        DOM.dragForm.classList.add('disabled');

        this.handleFiles(files);
      }
    },
    handleFiles: function(files) {
      ([...files]).forEach(this.uploadFile)
    },
    uploadFile: function(file) {
      const $this = this;
      const emoteSizes = [28,56,112];
      let modifiedImages = [];
      let imagesProcessed = 0;

      //Resize image
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
            $this.displayEmotes(sorted.reverse());
          }
        })
      })
    },
    displayEmotes: function(modifiedImages, size) {
      const $this = this;
      let imagesProcessed = 0;
      modifiedImages.forEach(function(imageBlob, index, array){
        //Create DOM elements
        const parent = document.createElement('div');
        const imageWrap = document.createElement('div');
        const size = document.createElement('p');
        const canvas = document.createElement('canvas');
        const btn = document.createElement('button');
        const context = canvas.getContext('2d');
        const baseImage = new Image();

        //Set image source to blob URL
        baseImage.src = window.URL.createObjectURL(imageBlob);

        baseImage.onload = function(){
          //Set canvas size
          canvas.height = baseImage.height;
          canvas.width = baseImage.width;

          //Draw canvas
          context.drawImage(baseImage, 0,0, canvas.width, canvas.height);

          //Conver to PNG
          const blobToPng = $this.convertToPng(canvas)

          //Set file name and bottom text to image width
          blobToPng.name = `${baseImage.width}.png`;
          size.textContent = `${baseImage.width} x ${baseImage.height}`;

          //Add image to zip (Image name, Image data)
          zip.file(blobToPng.name, imageBlob);

          //Increase image processed, if 3 generate zip
          imagesProcessed++;
          if(imagesProcessed === array.length) {
            zip.generateAsync({type:"blob"})
            .then(function(content) {

              $this.isActive = !$this.isActive;

              btn.classList.add('btn', 'shadow');
              btn.textContent = 'Download .zip';
              DOM.download.appendChild(btn);
              btn.removeEventListener('click', function(){});
              btn.addEventListener('click', function(){
                saveAs(content, `${Date.now()}.zip`);
              })
            });

            //Reenable form
            DOM.dragForm.classList.remove('disabled');
          }
        }

        //Apply classes and append to DOM
        parent.classList.add('result-wrap');
        imageWrap.classList.add('result', 'shadow');
        imageWrap.appendChild(canvas);
        parent.appendChild(imageWrap);
        parent.appendChild(size);
        DOM.results.appendChild(parent);
      })
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
      results: document.querySelector('.results'),
      download: document.querySelector('.download')
    }

    const events = ['dragenter', 'dragover', 'dragleave', 'drop'];

    //Prevent defaults
    events.forEach(event => {
      DOM.dragForm.addEventListener(event, function(e) {
        e.preventDefault()
        e.stopPropagation()
      }, false);
    })
  }
}
</script>
