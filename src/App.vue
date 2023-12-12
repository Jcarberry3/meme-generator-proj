<template>
  <div class="page-container" @dragover.prevent @drop.prevent>
    <div class="options-container">
        <OptionsWindow @draw-text="drawText" @crop-image="redrawCroppedImage"></OptionsWindow>
    </div> 
    <div class="content" v-cloak @drop.prevent="dropFile" @dragover.prevent>
        <h1>Drag and Drop your image here:</h1>
        <input type="file" accept="image/*" ref="fileInput" @change="uploadFile">
        <div id="canvasWrapper" class="canvasWrapper" ref="canvasWrapper">
            <canvas 
            id="imgCanvas" 
            ref="imgCanvas" 
            width="700" 
            height="700"
            ></canvas>
            <canvas 
            id="textCanvas"
            ref="textCanvas"
            width="700"
            height="700"
            @mousedown="handleMouseDown"
            @mousemove="handleMouseMove"
            @mouseup="handleMouseUp"
            @mouseout="handleMouseOut"
            ></canvas>
        </div>
                    <!-- Input fields for cropping the image -->

        <!-- <input type="number" class="cropWidth" v-model="cropLeft">
        <input type="number" class="cropWidth" v-model="cropRight">
        <input type="number" class="cropHeight" v-model="cropTop">
        <input type="number" class="cropHeight" v-model="cropBottom">
        <br> -->
                        <!-- Text Entry for meme text -->
            <!-- <input type="text" id="memeText" ref="memeText"> -->
            <!-- <button @click="drawText">Add Text</button> -->
            <a href="#" id="download-image-button" ref="downloadButton" download="my-file-name.png" @click="downloadImage">Download Image</a>
            <!-- <p>THIS IS TEST TEXT: {{ enteredMemeText }}</p> -->
    </div>
  </div>
</template>

<script>
import OptionsWindow from './components/OptionsWindow.vue';

export default {
    components: { OptionsWindow },
    // props: ['cropLeft', 'cropRight', 'cropTop', 'cropBottom', 'enteredMemeText'],
    data() {
        return {
            //internal references for both canvases
            vueImgCanvas: null,
            vueTextCanvas: null,

            // imgsrc: '',
            // testImage: require('../assets/MELEEFOX.jpg'),

            //assets for drawing image
            uploadedImage: '',
            // cropLeft: 0,
            // cropRight: 0,
            // cropTop: 0,
            // cropBottom: 0,

            //variables for mouse events
            offsetX: null,
            offsetY: null,
            scrollX: 0,
            scrollY: 0,
            //last mouse position
            startX: null,
            startY: null,

            //text
            memeTexts: [],
            selectedText: -1 //index of selected text from memeTexts array
        }
    },
    methods: {
        dropFile(e) {
            let file = e.dataTransfer.files[0];
            // let file = {...files}[0];
            if(!file || file.type.indexOf('image/') === -1) { return }

            let reader = new FileReader();

            reader.onload = (e) => { 
                    // this.imgsrc = e.target.result; //for testing with uploading to a regualr img element  
                    this.drawUploadedImage(e.target.result); 
                }
            reader.readAsDataURL(file);
        },
        uploadFile() {
            let file = this.$refs.fileInput.files[0];
            if(!file || file.type.indexOf('image/') === -1) { return }

            let reader = new FileReader();

            reader.onload = (e) => { this.imgsrc = e.target.result; }
            reader.readAsDataURL(file);
        },
        drawUploadedImage(imageSource) {
            // console.log('DRAWING UPLOADED IMAGE');
            this.vueImgCanvas.clearRect(0, 0, 700, 700);
            this.uploadedImage = new Image();
            
            this.uploadedImage.src = imageSource;
            this.uploadedImage.onload = () => { 
                this.vueImgCanvas.drawImage(this.uploadedImage, 0, 0); 
            }
            
        },
        redrawCroppedImage(newDimensions) {
            //Clear canvas:
            this.vueImgCanvas.clearRect(0, 0, 700, 700);

            //                   drawImage function
            // drawImage(image destX, destY)
            // drawImage(image, destX, destY, destWidth, destHeight)
            // drawImage(image, sourceX, sourceY, sourceWidth, sourceHeight, destX, destY, destWidth, destHeight)
            //                   drawImage function
            // 

            // let leftcrop = parseInt(this.cropLeft) || 0;
            // let rightcrop = parseInt(this.cropRight) || 0;
            // let topcrop = parseInt(this.cropTop) || 0;
            // let bottomcrop = parseInt(this.cropBottom) || 0;
            // let imgWidth = this.uploadedImage.width || 0;
            // let imgHeight = this.uploadedImage.height || 0;

            let leftcrop = parseInt(newDimensions.left) || 0;
            let rightcrop = parseInt(newDimensions.right) || 0;
            let topcrop = parseInt(newDimensions.top) || 0;
            let bottomcrop = parseInt(newDimensions.bottom) || 0;
            let imgWidth = this.uploadedImage.width || 0;
            let imgHeight = this.uploadedImage.height || 0;

            //                       Source Parameters
            // let sx = leftcrop;
            // let sy = topcrop;
            let width = imgWidth - (leftcrop + rightcrop); // sourceWidth
            let height = imgHeight - (topcrop + bottomcrop); // sourceHeight
            //                        Source Parameters

            //                      Destination Parameters
            // let dx = leftcrop + imgWidth;
            // let dy = topcrop + imgHeight;
            // let destWidth = imgWidth;
            // let destHeight = imgHeight;
            //                      Destination Parameters

            this.vueImgCanvas.drawImage(this.uploadedImage, 
            leftcrop,   //sourceX
            topcrop,    //sourceY
            width,      //sourceWidth
            height,     //sourceHeight
            leftcrop,         //destinationX
            topcrop,         //destinationY
            width,      //destinationWidth
            height      //destinationHeight
            );
        },
        handleMouseDown(e) {
            console.log('MOUSE DOWN DETECTED!') //DEBUG
            e.preventDefault();
            this.startX = parseInt(e.clientX - this.offsetX);
            this.startY = parseInt(e.clientY - this.offsetY);
            console.log('MouseX: ' + this.startX); //DEBUG
            console.log('MouseY: ' + this.startY); //DEBUG
            for(var i = 0; i < this.memeTexts.length; i++)
            {
                if(this.textHitTest(this.startX, this.startY, i))
                {
                    console.log('TEXT COLLISION DETECTED!') //DEBUG
                    this.selectedText = i;
                }
            }
        },
        //mouseDown event helper functions
        textHitTest(x, y, textIndex) {
            console.log('Checking for text collision...');
            var text = this.memeTexts[textIndex];
            return x <= text.x + text.width && y >= text.y - text.height && y <= text.y;
        },
        //mouseDown event helper functions
        handleMouseMove(e) {
            // console.log('MOUSE MOVE DETECTED!') //DEBUG
            if(this.selectedText < 0) { return; }
            e.preventDefault();
            let mouseX = parseInt(e.clientX - this.offsetX);
            let mouseY = parseInt(e.clientY - this.offsetY);

            var dx = mouseX - this.startX;
            var dy = mouseY - this.startY;
            this.startX = mouseX;
            this.startY = mouseY;

            var text = this.memeTexts[this.selectedText];
            text.x += dx;
            text.y += dy;
            this.draw();
        },
        handleMouseUp(e) {
            e.preventDefault();
            this.selectedText = -1;
        },
        handleMouseOut(e) {
            e.preventDefault();
            this.selectedText = -1;
        },
        draw() {
            this.vueTextCanvas.clearRect(0, 0, this.$refs.textCanvas.width, this.$refs.textCanvas.height);
            for(var i = 0; i < this.memeTexts.length; i++)
            {
                var text = this.memeTexts[i];
                this.vueTextCanvas.fillText(text.text, text.x, text.y);
            }
        },
        drawText(textObj) {
            //calc the y coordinate for this text on the canvas
            var y = this.memeTexts.length * 20 + 20;

            //get the text from the input element
            var text = { 
                // text: this.enteredMemeText,
                text: textObj.text,
                x: 20,
                y: y
            };

            //calc the size of this text for hit-testing purposes
            this.vueTextCanvas.font = '28px verdana';
            this.vueTextCanvas.fillStyle = textObj.color;
            text.width = this.vueTextCanvas.measureText(text.text).width;
            text.height = 16;

            //put the new text in the memeTexts array
            this.memeTexts.push(text);

            //redraw canvas
            this.draw();

            //DEBUG
            // console.log(this.memeTexts[0]);
            // console.log('X: ' + text.x);
            // console.log('Y: ' + text.y);
        },
        downloadImage() {
            //'flatten' canvas layers
            this.vueImgCanvas.drawImage(this.$refs.textCanvas, 0, 0);
            //download image
            var canvas = this.$refs.imgCanvas;
            var button = this.$refs.downloadButton;
            var dataURL = canvas.toDataURL('image/png');
            button.href = dataURL;
        }
        //                                                  DEBUG
        // drawTestImage() { 
        //     console.log('DRAWING TEST IMAGE');
        //     // var img = this.$refs.testImage;
        //     var img = new Image();
        //     img.src = require('../assets/MELEEFOX.jpg');
        //     img.onload = () => {
        //         this.vueImgCanvas.drawImage(img, 10, 10);
        //     }
             

        //     // this.vueImgCanvas.drawImage(this.testImage, 10, 10)

        //     // DRAW TEST RECT
        //     // this.vueImgCanvas.beginPath();
        //     // this.vueImgCanvas.rect(20, 20, 200, 100);
        //     // this.vueImgCanvas.stroke();
        // },
        // drawActualImage(image) {
        //     this.vueImgCanvas.drawImage(image, 10, 10);
        // }
    },
    mounted() {
        //define offsetX and offsetY
        this.offsetX = this.$refs.canvasWrapper.offsetLeft;
        this.offsetY = this.$refs.canvasWrapper.offsetTop;

        //const c = document.getElementById('imgCanvas'); // this is the js alternative to the next line
        const c = this.$refs.imgCanvas;
        this.vueImgCanvas = c.getContext('2d');

        const c2 = this.$refs.textCanvas;
        this.vueTextCanvas = c2.getContext('2d');
    },
    watch: { //watch for a change in any of the dimentions and redraw the image
        cropLeft() {
            this.redrawCroppedImage();
        },
        cropRight() {
            this.redrawCroppedImage();
        },
        cropTop() {
            this.redrawCroppedImage();
        },
        cropBottom() {
            this.redrawCroppedImage();
        }
    }
}
</script>

<style lang="less">
* { //CSS reset
  position: block;
  margin: 0;
  padding: 0;
  border: 0;
  outline: 0;
  font-size: 100%;
  vertical-align: baseline;
  background: transparent;

  color: white;
}

canvas {
    position: absolute;
    // top: 0;
    // left: 10;
    border: solid black 1px;
}

#memeText {
    //DEBUG
    border: solid black 1px;
    background-color: white;
}

#download-image-button {
    background-color: #3a3b3c;
}

.canvasWrapper {
    position: relative;



    width: 700px;
    height: 700px;
    //DEBUG
    border: solid black 1px;
    background-color: #3a3b3c;
}

// .cropWidth {
//     //DEBUG
//     border: solid black 1px;
//     background-color: teal;
// }
// .cropHeight {
//     //DEBUG
//     border: solid black 1px;
//     background-color: pink;
// }

button {
    background-color: aqua;
    box-shadow: 2px 2px;
    border-radius: 5px;

    &:focus {
        background-color: blue;
    }

    &:active {
        background-color: red;
    }
}
.content {
    height: 80%;
    //DEBUG
    border: solid black 1px;

    .file-preview {
        max-width: 250px;
        max-height: 250px;

        //DEBUG
        border: solid black 1px;
    }
}

.page-container {
  display: flex;
  flex-direction: row;

  width: 100%;
  height: 100vh;
  /* //DEBUG */
//   background-color: yellow;

  .options-container {
    flex-grow: .25;

    height: 100%;
    max-width: 270px;

    //DEBUG
    // border: solid black 1px;
    background-color: #18191a;
  }
  
  .content {
    flex-grow: 1;

    height: 100%;

    padding-left: 500px;

    //DEBUG
    // border: solid black 1px;
    background-color: #18191a;
  }
}

</style>