<template>
  <div class="options-window">
    Crop from (in pxls):<br>
    <div>Left: <input type="number" class="cropWidth" v-model="left" @input="changeCropping"></div>
    <div>Right: <input type="number" class="cropWidth" v-model="right" @input="changeCropping"></div>
    <div>Top: <input type="number" class="cropHeight" v-model="top" @input="changeCropping"></div>
    <div>Bottom: <input type="number" class="cropHeight" v-model="bottom" @input="changeCropping"></div>
    <input type="text" id="memeText" ref="memeText" placeholder="add some text...">
    
    <input type="radio" id="black" name="color" value="black"> <label for="black">Black</label>
    <input type="radio" id="white" name="color" value="white"> <label for="white">White</label>
    
    <button @click="saveEnteredText" style="color: black">Add Text</button>
    </div>
    <!-- <div class="content-container">
      <OptionsWindow
      :cropLeft="left"
      :cropRight="right"
      :cropTop="top"
      :cropBottom="bottom"
      :enteredMemeText="textInput"
      ></OptionsWindow>
    </div> -->
</template>

<script>
export default {
  data() {
    return {
      left: 0,
      right: 0,
      top: 0,
      bottom: 0,

      textInput: ''
    }
  },
  methods: {
    saveEnteredText() {
      this.textInput = this.$refs.memeText.value;

      let color = '';
      let radios = document.getElementsByName('color');
      radios.forEach((i) => { 
        if(i.checked) 
        {
          color = i.value;
        }
        });
      let textObj = {
        text: this.textInput,
        color: color
      }
      this.$emit('draw-text', textObj);
    },
    changeCropping() {
      let newDimensions = {
        left: this.left,
        right: this.right,
        top: this.top,
        bottom: this.bottom
      }
      this.$emit('crop-image', newDimensions)
    }
  }
}
</script>

<style lang="less" scoped>



.options-window {
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  height: 250px;

  background-color: #242526;
  color: white;
  .cropWidth {
    //DEBUG
    border: solid black 1px;
    background-color: teal;
  }
  .cropHeight {
    //DEBUG
    border: solid black 1px;
    background-color: pink;
  }
}

// .page-container {
//   display: flex;
//   flex-direction: row;

//   width: 100vw;
//   height: 100vh;
//   /* //DEBUG */
//   // background-color: yellow;

//   .options-container {
//     flex-grow: .25;

//     height: 100%;

//     //DEBUG
//     // border: solid black 1px;
//     background-color: red;
//   }
  
//   .content-container {
//     flex-grow: .75;

//     height: 100%;

//     //DEBUG
//     // border: solid black 1px;
//     background-color: green;
//   }
// }
</style>
