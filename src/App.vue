<template>
  <div class="container">
    <select name="selection" id="selection">
        <option v-for="i in inputTypes.length" :key="i-=1" v-bind:value="inputTypes[i]"> {{inputNames[i]}} </option>
    </select>

    <input type="button" value="Add Input" @click="addInput();">

    <div class="list">
      <ol>
        <div v-for="i in inputsArray.length" :key="i">
          <li v-bind:id="i">
            <div>{{inputSelectedNames[i-1]}} 
              <button v-bind:id="i" @click="switchDown(inputSelectedNames,$event); switchDown(inputsArray,$event)">&darr;</button>
               <button v-bind:id="i" @click="switchUp(inputSelectedNames,$event); switchUp(inputsArray,$event)">&uarr;</button>
                <input v-if="inputSelectedNames[i-1] != 'Hidden Field'" v-bind:id="i" type="text" placeholder="Please enter a Name" @change="changeName($event)">
                <button @click="deleteElement($event)" v-bind:id="i">Delete</button></div>
          </li>
        </div>
      </ol>
    </div>

    <div v-if="inputsArray.length != 0">
      <button @click="viewPreview();">Show Preview</button>
    </div>

    <div v-if="showPreview && inputsArray.length != 0" id="preview" >
      <button @click="download();">Download</button>

      <div id="color_picker">
        <input id="color_input_bg" type="color" >
        <input type="number" min="0.0" max="1.0" step="0.1" id="alpha_bg" style="width: 6%" placeholder="Alpha">
        <button @click="pickColorBG();">Change Background Color</button>
      </div>

      <div id="color_picker">
        <input id="color_input_text" type="color" >
        <input type="number" min="0.0" max="1.0" step="0.1" id="alpha_text" style="width: 6%" placeholder="Alpha">
        <button @click="pickColorText();">Change Text Color</button>
      </div>

      <div id="form_preview">
        <form id="my_form" action="#" v-bind:style="form_style">
          <div v-for="i in inputsArray.length" :key="i">
            <div><label v-if="inputSelectedNames[i-1] != 'Hidden Field' && inputSelectedNames[i-1] != 'Submit'" for="" > {{inputs_labelNames[i-1]}} </label></div>
            <div v-html="inputsArray[i-1]">
            </div>
          </div>
        </form>
      </div>
    
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line no-unused-vars
import axios from 'axios'


export default {
  name: 'App',
  components: {
    
  },
  data() {
    return {
      showPreview: false,
      color_bg: 'rgba(84, 181, 255, 0.3)',
      color_text: '#000000',
      form_style: {backgroundColor: this.color_bg,
      color: this.color_text},
      labelBoxVals: [],




      inputsArray: [],
      inputsValues: [],
      inputSelectedNames: [],
      inputTypes:['name','password','file_pdf','file_img','hidden','text','submit'],
      inputNames: ['Name','Password','PDF File','Image','Hidden Field','Text','Submit'],
      labelRawHTML: '<label class="form-label" for="name">Enter your name:</label>',
      inputsRawHTML: [],
      inputsRawHTML_: 
      {//hidden
        'name': `<input class="form-control" type="text" name="name" id="name">`,
      //password
        'password': `<input class="form-control" type="password" name="password" id="password">`,
      //image file
      'file_img': `
            <input class="form-control" type="file" accept="image/*" name="img" id="img">`,
      //pdf file
      'file_pdf': `<input class="form-control" type="file" accept="application/pdf" name="pdf" id="pdf">`,
      //text field
        'text': `<input class="form-control" type="text" name="text_field" id="text_field">`,
      //hidden
        'hidden': `<input type="hidden" name="hidden" id="hidden">`,
      //submit button
        'submit': `<input class="btn btn-primary" type="button" name="submit" id="submit" value="Submit">`
      },
      inputs_labelNames: [],
      final: ''
    }
  },
  methods : {
    viewPreview: function (){
      this.showPreview = !this.showPreview;
    },
    addInput: function(){
      var selection = document.getElementById('selection')
      var s = selection.value
      var n = selection.options[selection.options.selectedIndex].innerHTML
      this.inputSelectedNames.push(n);
      this.inputsValues.push(s);
      this.inputsArray.push(this.inputsRawHTML_[s]);
      this.inputs_labelNames.push('Please enter a Name')
    },
    pickColorBG: function (){
      var picked_color = document.getElementById('color_input_bg').value;
      var alpha = document.getElementById('alpha_bg').value;
      var rgba = this.hexToRGB(picked_color,alpha);
      this.color_bg = rgba
      console.log(this.color_bg + '=' + picked_color);
    },
    pickColorText: function (){
      var picked_color = document.getElementById('color_input_text').value;
      var alpha = document.getElementById('alpha_text').value;
      var rgba = this.hexToRGB(picked_color,alpha);
      this.color_text = rgba
      console.log(this.color_text + '=' + picked_color);
    },
    hexToRGB: function(hex, alpha) {
      var r = parseInt(hex.slice(1, 3), 16),
          g = parseInt(hex.slice(3, 5), 16),
          b = parseInt(hex.slice(5, 7), 16);

      if (alpha) {
          return "rgba(" + r + ", " + g + ", " + b + ", " + alpha + ")";
      } else {
          return "rgb(" + r + ", " + g + ", " + b + ")";
      }
    },
    switchUp: function(array,event){
      var i = event.target.id;
      if(i != parseInt(1)){
        i-=1;
        var temp = array[i];
        array[i] = array[i-1];
        array[i-1] = temp;
      }
    },
    switchDown: function(array,event){
      var i = event.target.id;
      if(i != array.length){
        i-=1;
        var temp = array[i];
        array[i] = array[i+1];
        array[i+1] = temp;
      }
    },
    deleteElement(event){
      var i = parseInt(event.target.id) - 1;
      console.log(i);
      this.inputsArray.splice(i,1)
      this.inputsValues.splice(i,1)
      this.inputSelectedNames.splice(i,1)
      this.inputsRawHTML.splice(i,1)
      this.labelBoxVals.splice(i,1)
      this.inputs_labelNames.splice(i,1)
    },
    changeName: function (event){
    var index = parseInt(event.target.id) - 1;
    var name = event.target.value;
    this.inputs_labelNames[index] = name;
    },
    getFormRawHTML: function(){
      var text = document.getElementById('form_preview').innerHTML
      var regex = /<!--v-if-->/g;
      text.replace(regex,'');
      return text;
    },
    download: async function(){
      var form = this.getFormRawHTML();
      var style = `
      <style>
        #my_form {
          background-color: `+this.color_bg+`;
          color: `+this.color_text+`;
          width: 50%;
          margin-top: 5%;
          margin-left: auto;
          margin-right: auto;

          padding: 2%;

          border-radius: 25px;
        </style>
        }`

        var final = form+style;

        this.final = final;

        axios.post('http://localhost/form_gen_server/download.php',{html: this.final},)
          .then(function (response) {
            console.log(final)
            console.log(response);
        })
          .catch(function (error) {
          console.log(error);
        }
      );



      },
      downloadLink: function(filename, text) {
        var element = document.createElement('a');
        element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
        element.setAttribute('download', filename);

        element.style.display = 'none';
        document.body.appendChild(element);

        element.click();

        document.body.removeChild(element);
      }
    }
}
</script>

<style>

#my_form {
    background-color: v-bind('color_bg');
    color: v-bind('color_text');
    width: 50%;
    margin-top: 5%;
    margin-left: auto;
    margin-right: auto;

    padding: 2%;

    border-radius: 25px;
  }

</style>
