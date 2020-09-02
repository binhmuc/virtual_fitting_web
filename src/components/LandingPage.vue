<template>
  <div id="wrapper">
    <h1>
      Virtual Fitting
    </h1>
    <div class="container">
      <div class="section">
        <h3>Person image</h3>
        <div class="row">
            <div class="col-md-6">
                <code v-if="device">{{ device.label }}</code>
                <div class="border">
                    <vue-web-cam
                        ref="webcam"
                        :device-id="deviceId"
                        :resolution="resolution"
                        :height="videoHeight"
                        :width="videoWidth"
                        @started="onStarted"
                        @stopped="onStopped"
                        @error="onError"
                        @cameras="onCameras"
                        @camera-change="onCameraChange"
                    />
                </div>

                <div class="row">
                    <div class="col-md-12">
                        <select v-model="camera">
                            <option>-- Select Device --</option>
                            <option
                                v-for="device in devices"
                                :key="device.deviceId"
                                :value="device.deviceId"
                            >{{ device.label }}</option>
                        </select>
                    </div>
                    <div class="col-md-12">
                        <button type="button" class="btn btn-primary" @click="onCapture">Capture Photo</button>
                        <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
                        <button type="button" class="btn btn-success" @click="onStart">Start Camera</button>
                        
                        <form id="upload-file"
                            method="post"
                            enctype="multipart/form-data">
                        <label for="imageUpload"
                              class="upload-label">
                          Upload image...
                        </label>
                        <input type="file"
                              name="file"
                              @change="contentUpload"
                              id="imageUpload"
                              accept=".jpg">
                        </form>

                        <br/>
                        <div class="image-container">
                          <div class="image-flex">
                            <div v-for="image in personImages"
                                 :key="image.id"
                                 class="image-item">
                              <image-item :src="image.src"
                                          :id="image.id"
                                          :selected="selectedPersonId"
                                          @clicked="onSelectPerson"></image-item>
                            </div>
                          </div>
                        </div>
                        <div class="style-hint">
                          Choose a person
                        </div>

                    </div>
                </div>
            </div>
        </div>
      </div>

      <div class="section">
        <h3>Clothes</h3>
        <div class="image-container">
          <div class="image-flex">
            <div v-for="image in clothImages"
                 :key="image.id"
                 class="image-item">
              <image-item :src="image.src"
                          :id="image.id"
                          :selected="selectedClothId"
                          @clicked="onSelectCloth"></image-item>
            </div>
          </div>
        </div>

        <div class="padding">
          <toggle-button @change="toggleResult" 
           :labels="{checked: 'Upper', unchecked: 'Pants'}"
           :color="{checked: '#BE3D62', unchecked: '#00A388'}"
           :width="70"
          /> 
        </div>

        <div class="style-hint">
          Choose a cloth OR
          <form id="upload-style"
                method="post"
                enctype="multipart/form-data">
            <label for="styleUpload"
                   class="upload-label">
              upload yours
            </label>
            <input type="file"
                   name="file"
                   @change="styleUpload"
                   id="styleUpload"
                   accept=".png, .jpg, .jpeg">
          </form>
        </div>

        <div v-if="userStyle"
             class="upload-style">
          
          <img :src="userStyleSrc"
               alt="">
          <div class="remove"
               @click="removeUserStyle">Remove
          </div>
        </div>

        <button class="btn"
                :disabled="submitDisable"
                @click="submitDrawing">
          <span class="submit"></span>
          <span>Submit</span>
        </button>

      </div>

      <div class="section">
        <h3>Result</h3>
        <div class="result-container">
          <div class="hint"
               v-if="!resultSrc">
            Your result will be shown here.
          </div>
          <img v-if="resultSrc"
               :src="resultSrc"
               alt="">
        </div>
        <div class="col-md-6">
                <h2>Captured Image</h2>
                <figure class="figure">
                    <img :src="img" height="420" width="315" class="img-responsive" />
                </figure>
        </div>
      </div>
    </div>

    <div class="overlay"
         v-if="showWaitModal">
      <div class="half-circle-spinner">
        <div class="circle circle-1"></div>
        <div class="circle circle-2"></div>
      </div>
      <div class="content">
        {{ modalContent }}
      </div>
    </div>
  </div> 
</template>

<script>
import DrawingBoard from "./DrawingBoard.vue";
import ImageItem from "./ImageItem.vue";
import axios from "axios";
import Vue from "vue";
import VueSwal from "vue-swal";
import ToggleButton from "vue-js-toggle-button";
import Webcam from "vue-web-cam";

Vue.use(Webcam);
Vue.use(VueSwal);
Vue.use(ToggleButton);

// const axiosVirtualFit = axios.create({ baseURL: "https://192.245.249.149:5002/" });
const axiosVirtualFit = axios.create({ baseURL: process.env.BASE_API });

const type = {
  "000048_1.jpg": "short",
  "000118_1.jpg": "long",
  "000228_1.jpg": "short",
  "002362_1.jpg": "long",
  "003124_1.jpg": "vest",
  "003414_1.jpg": "vest",
  "004462_1.jpg": "short",
  "007345_1.jpg": "short",
  "008451_1.jpg": "long",
  "008023_1.jpg": "vest",
  "008570_1.jpg": "short",
  "014159_1.jpg": "short",
  "011615_1.jpg": "vest",
  "011798_1.jpg": "vest",
  "003414_1.jpg": "vest",
  "001664_1.jpg": "vest",
  "003434_1.jpg": "long"
};

function resizedataURL(base64String, wantedWidth, wantedHeight){
  return new Promise(async function(resolve,reject){
    var img = document.createElement('img');
    img.onload = function()
    {        
        var canvas = document.createElement('canvas');
        var ctx = canvas.getContext('2d');
        canvas.width = wantedWidth;
        canvas.height = wantedHeight;
        ctx.drawImage(this, 0, 0, wantedWidth, wantedHeight);
        var dataURI = canvas.toDataURL('image/jpeg', 1.0);
        resolve(dataURI);
    };
    img.src = base64String;
  })
}

function dectectMob() {
  if( navigator.userAgent.match(/Android/i)
     || navigator.userAgent.match(/webOS/i)
     || navigator.userAgent.match(/iPhone/i)
     || navigator.userAgent.match(/iPad/i)
     || navigator.userAgent.match(/iPod/i)
     || navigator.userAgent.match(/BlackBerry/i)
     || navigator.userAgent.match(/Windows Phone/i)
  ) {
    return true;
  }
  else {
    return false;
  }
}

export default {
  name: "LandingPage",

  components: {
    DrawingBoard,
    ImageItem,
    Webcam
  },

  data() {
    return {
      toggled: false,
      msg: "Welcome",
      personImages: [
       { id: "1.jpg", src: require("@/assets/persons/1.jpg") },
        { id: "2.jpg", src: require("@/assets/persons/2.jpg") },
        { id: "3.jpg", src: require("@/assets/persons/3.jpg") },
        { id: "4.jpg", src: require("@/assets/persons/4.jpg") },
        { id: "5.jpg", src: require("@/assets/persons/5.jpg") },
        { id: "6.jpg", src: require("@/assets/persons/6.jpg") },
        { id: "7.jpg", src: require("@/assets/persons/7.jpg") },
        { id: "8.jpg", src: require("@/assets/persons/8.jpg") },
        { id: "9.jpg", src: require("@/assets/persons/9.jpg") },
        { id: "10.jpg", src: require("@/assets/persons/10.jpg") },
        { id: "11.jpg", src: require("@/assets/persons/11.jpg") },
        { id: "12.jpg", src: require("@/assets/persons/12.jpg") },
        { id: "13.jpg", src: require("@/assets/persons/13.jpg") },
        { id: "14.jpg", src: require("@/assets/persons/14.jpg") },
        { id: "15.jpg", src: require("@/assets/persons/15.jpg") },
         { id: "16.jpg", src: require("@/assets/persons/16.jpg") },
        { id: "17.jpg", src: require("@/assets/persons/17.jpg") },
        { id: "18.jpg", src: require("@/assets/persons/18.jpg") },
        { id: "19.jpg", src: require("@/assets/persons/19.jpg") },
        { id: "20.jpg", src: require("@/assets/persons/20.jpg") },
      ],
      p1: [
      { id: "1.jpg", src: require("@/assets/persons/1.jpg") },
        { id: "2.jpg", src: require("@/assets/persons/2.jpg") },
        { id: "3.jpg", src: require("@/assets/persons/3.jpg") },
        { id: "4.jpg", src: require("@/assets/persons/4.jpg") },
        { id: "5.jpg", src: require("@/assets/persons/5.jpg") },
        { id: "6.jpg", src: require("@/assets/persons/6.jpg") },
        { id: "7.jpg", src: require("@/assets/persons/7.jpg") },
        { id: "8.jpg", src: require("@/assets/persons/8.jpg") },
        { id: "9.jpg", src: require("@/assets/persons/9.jpg") },
        { id: "10.jpg", src: require("@/assets/persons/10.jpg") },
        { id: "11.jpg", src: require("@/assets/persons/11.jpg") },
        { id: "12.jpg", src: require("@/assets/persons/12.jpg") },
        { id: "13.jpg", src: require("@/assets/persons/13.jpg") },
        { id: "14.jpg", src: require("@/assets/persons/14.jpg") },
        { id: "15.jpg", src: require("@/assets/persons/15.jpg") },
        { id: "16.jpg", src: require("@/assets/persons/16.jpg") },
        { id: "17.jpg", src: require("@/assets/persons/17.jpg") },
        { id: "18.jpg", src: require("@/assets/persons/18.jpg") },
        { id: "19.jpg", src: require("@/assets/persons/19.jpg") },
        { id: "20.jpg", src: require("@/assets/persons/20.jpg") },
      ],
      p2: [
        { id: "1.jpg", src: require("@/assets/p_persons/1.jpg") },
        { id: "2.jpg", src: require("@/assets/p_persons/2.jpg") },
        { id: "3.jpg", src: require("@/assets/p_persons/3.jpg") },
        { id: "4.jpg", src: require("@/assets/p_persons/4.jpg") },
        { id: "5.jpg", src: require("@/assets/p_persons/5.jpg") },
        { id: "6.jpg", src: require("@/assets/p_persons/6.jpg") },
        { id: "7.jpg", src: require("@/assets/p_persons/7.jpg") },
        { id: "8.jpg", src: require("@/assets/p_persons/8.jpg") },
        { id: "9.jpg", src: require("@/assets/p_persons/9.jpg") },
        { id: "10.jpg", src: require("@/assets/p_persons/10.jpg") },
        { id: "11.jpg", src: require("@/assets/p_persons/11.jpg") },
        { id: "12.jpg", src: require("@/assets/p_persons/12.jpg") },
        { id: "13.jpg", src: require("@/assets/p_persons/13.jpg") },
        { id: "14.jpg", src: require("@/assets/p_persons/14.jpg") },
        { id: "15.jpg", src: require("@/assets/p_persons/15.jpg") },
        { id: "16.jpg", src: require("@/assets/p_persons/16.jpg") },
        { id: "17.jpg", src: require("@/assets/p_persons/17.jpg") },
        { id: "18.jpg", src: require("@/assets/p_persons/18.jpg") },
        { id: "19.jpg", src: require("@/assets/p_persons/19.jpg") },
        { id: "20.jpg", src: require("@/assets/p_persons/20.jpg") },
      ],
      clothImages: [
        { id: "1.jpg", src: require("@/assets/cloths/1.jpg") },
        { id: "2.jpg", src: require("@/assets/cloths/2.jpg") },
        { id: "3.jpg", src: require("@/assets/cloths/3.jpg") },
        { id: "4.jpg", src: require("@/assets/cloths/4.jpg") },
        { id: "5.jpg", src: require("@/assets/cloths/5.jpg") },
        { id: "6.jpg", src: require("@/assets/cloths/6.jpg") },
        { id: "7.jpg", src: require("@/assets/cloths/7.jpg") },
        { id: "8.jpg", src: require("@/assets/cloths/8.jpg") },
        { id: "9.jpg", src: require("@/assets/cloths/9.jpg") },
        { id: "10.jpg", src: require("@/assets/cloths/10.jpg") },
        { id: "11.jpg", src: require("@/assets/cloths/11.jpg") },
        { id: "12.jpg", src: require("@/assets/cloths/12.jpg") },
        { id: "13.jpg", src: require("@/assets/cloths/13.jpg") },
        { id: "14.jpg", src: require("@/assets/cloths/14.jpg") },
        { id: "15.jpg", src: require("@/assets/cloths/15.jpg") },
        { id: "16.jpg", src: require("@/assets/cloths/16.jpg") },
        { id: "17.jpg", src: require("@/assets/cloths/17.jpg") },
        { id: "18.jpg", src: require("@/assets/cloths/18.jpg") },
        { id: "19.jpg", src: require("@/assets/cloths/19.jpg") },
        { id: "20.jpg", src: require("@/assets/cloths/20.jpg") },
        { id: "21.jpg", src: require("@/assets/cloths/21.jpg") },
      ],
      clothes: [
        { id: "1.jpg", src: require("@/assets/cloths/1.jpg") },
        { id: "2.jpg", src: require("@/assets/cloths/2.jpg") },
        { id: "3.jpg", src: require("@/assets/cloths/3.jpg") },
        { id: "4.jpg", src: require("@/assets/cloths/4.jpg") },
        { id: "5.jpg", src: require("@/assets/cloths/5.jpg") },
        { id: "6.jpg", src: require("@/assets/cloths/6.jpg") },
        { id: "7.jpg", src: require("@/assets/cloths/7.jpg") },
        { id: "8.jpg", src: require("@/assets/cloths/8.jpg") },
        { id: "9.jpg", src: require("@/assets/cloths/9.jpg") },
        { id: "10.jpg", src: require("@/assets/cloths/10.jpg") },
        { id: "11.jpg", src: require("@/assets/cloths/11.jpg") },
        { id: "12.jpg", src: require("@/assets/cloths/12.jpg") },
        { id: "13.jpg", src: require("@/assets/cloths/13.jpg") },
        { id: "14.jpg", src: require("@/assets/cloths/14.jpg") },
        { id: "15.jpg", src: require("@/assets/cloths/15.jpg") },
        { id: "16.jpg", src: require("@/assets/cloths/16.jpg") },
        { id: "17.jpg", src: require("@/assets/cloths/17.jpg") },
        { id: "18.jpg", src: require("@/assets/cloths/18.jpg") },
        { id: "19.jpg", src: require("@/assets/cloths/19.jpg") },
        { id: "20.jpg", src: require("@/assets/cloths/20.jpg") },
        { id: "21.jpg", src: require("@/assets/cloths/21.jpg") },
      ],

      pants: [
       { id: "1.jpg", src: require("@/assets/pants/1.jpg") },
        { id: "2.jpg", src: require("@/assets/pants/2.jpg") },
        { id: "3.jpg", src: require("@/assets/pants/3.jpg") },
        { id: "4.jpg", src: require("@/assets/pants/4.jpg") },
        { id: "5.jpg", src: require("@/assets/pants/5.jpg") },
        { id: "6.jpg", src: require("@/assets/pants/6.jpg") },
        { id: "7.jpg", src: require("@/assets/pants/7.jpg") },
        { id: "8.jpg", src: require("@/assets/pants/8.jpg") },
        { id: "9.jpg", src: require("@/assets/pants/9.jpg") },
        { id: "10.jpg", src: require("@/assets/pants/10.jpg") },
        { id: "11.jpg", src: require("@/assets/pants/11.jpg") },
        { id: "12.jpg", src: require("@/assets/pants/12.jpg") },
        { id: "13.jpg", src: require("@/assets/pants/13.jpg") },
        { id: "14.jpg", src: require("@/assets/pants/14.jpg") },
        { id: "15.jpg", src: require("@/assets/pants/15.jpg") },
        { id: "16.jpg", src: require("@/assets/pants/16.jpg") },
        { id: "17.jpg", src: require("@/assets/pants/17.jpg") },
        { id: "18.jpg", src: require("@/assets/pants/18.jpg") },
        { id: "19.jpg", src: require("@/assets/pants/19.jpg") },
        { id: "20.jpg", src: require("@/assets/pants/20.jpg") },
        { id: "21.jpg", src: require("@/assets/pants/21.jpg") },
      ],

      dataUri: null,
      resolution: (dectectMob())? { height: 720, width: 960 }: { height: 960, width: 720 },
      selectedPersonId: null,
      selectedClothId: null,
      sessionId: "",
      personImage: null,
      clothImage: null,
      userContent: false,
      userStyle: false,
      userStyleSrc: "",

      highReality: true,
      highQuality: false,

      showStyle: true,
      showToggle: false,
      resultSrc: "",
      resultPix: "",
      resultStyle: "",

      showWaitModal: false,
      modalContent: "Waiting for a few seconds...",
      submitDisable: true,

      img: null,
      camera: null,
      deviceId: null,
      devices: [],
      videoHeight: 448,
      videoWidth: 336
    };
  },

  computed: {
      device: function() {
          return this.devices.find(n => n.deviceId === this.deviceId);
      }
  },

  watch: {
      camera: function(id) {
          this.deviceId = id;
      },
      devices: function() {
          // Once we have a list select the first one
          const [first, ...tail] = this.devices;
          if (first) {
              this.camera = first.deviceId;
              this.deviceId = first.deviceId;
          }
      }
  },

  mounted: function() {
    this.selectedId = 1;

    let that = this;
    axiosVirtualFit
      .get("/")
      .catch(function(error) {
        that.$swal({
          title: "Something wrong...",
          text:
            "Server not responding, check your Internet connection first. You can view details in the About page. To experience the app, please contact the author for help.",
          icon: "warning",
          buttons: {
            cancel: "Got it"
          }
        });
        console.log("virtual fitting server error");
      })
      .then(response => {
        console.log("Connected to virtual fitting server...");
      });
  },

  methods: {
    clearCanvas() {
      this.$refs.canvas.clearCanvas();
      this.userContent = false;
    },

    onSelectPerson(id) {
      this.selectedPersonId = id;
      this.img = this.personImages.find(obj => obj.id === id).src;
      this.submitDisable = false;
    },

    onSelectCloth(id) {
      this.selectedClothId = id;
      this.clothImage = this.clothImages.find(obj => obj.id === id).src;
      this.submitDisable = false;
    },
    submitDrawing() {
      var vfData = new FormData();
      vfData.append("person_image", this.img);
      vfData.append("cloth_image", this.clothImage);
      // vfData.append("type", type[this.selectedClothId]);

      this.modalContent = "Waiting for a few seconds...";
      this.showWaitModal = true;
      if (!this.toggled) {
        axiosVirtualFit({
          url: "/virtual_fitting",
          method: "POST",
          data: vfData,
          headers: {
            "Content-Type": "multipart/form-data"
          }
        })
        .then(response => {
          this.showWaitModal = false;
          this.resultSrc = response.data;
        });
      } else {
        axiosVirtualFit({
          url: "/virtual_fitting_bottom",
          method: "POST",
          data: vfData,
          headers: {
            "Content-Type": "multipart/form-data"
          }
        })
        .then(response => {
          this.showWaitModal = false;
          this.resultSrc = response.data;
        });
      }
    },

    toggleResult() {
      this.toggled = !this.toggled;
      if (!this.toggled) {
        this.personImages = this.p1;
        this.clothImages = this.clothes;
      } else {
        this.personImages = this.p2;
        this.clothImages = this.pants;
      }
    },
    async onCapture() {
      this.img = this.$refs.webcam.capture();
    },
    onStarted(stream) {
        console.log("On Started Event", stream);
    },
    onStopped(stream) {
        console.log("On Stopped Event", stream);
    },
    onStop() {
        this.$refs.webcam.stop();
    },
    onStart() {
        this.$refs.webcam.start();
    },
    onError(error) {
        console.log("On Error Event", error);
    },
    onCameras(cameras) {
        this.devices = cameras;
        console.log("On Cameras Event", cameras);
    },
    onCameraChange(deviceId) {
        this.deviceId = deviceId;
        this.camera = deviceId;
        console.log("On Camera Change Event", deviceId);
    },
    contentUpload(e) {
      var files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;
      this.personImage = files[0];
      var reader = new FileReader();
      reader.readAsDataURL(files[0]);
      reader.onload = async(e) => {
        this.img = e.target.result;
      };
    },
    styleUpload(e) {
      var files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;
      var reader = new FileReader();
      var vm = this;
      reader.onload = e => {
        this.clothImage = e.target.result;
        var canvas = document.createElement("canvas");
        var ctx = canvas.getContext("2d");
        var image = new Image();
        canvas.width = 192;
        canvas.height = 256;
        var imgData;
        image.onload = function() {
          ctx.drawImage(image, 0, 0, 192, 256);
          let imgData = canvas.toDataURL("image/jpg");
          vm.setUserStyleSrc(imgData);
        };
        image.src = e.target.result;
      };
      reader.readAsDataURL(files[0]);
      e.target.value = "";
    },
    setUserStyleSrc(data) {
      this.userStyleSrc = data;
      this.userStyle = true;
      this.submitDisable = false;
      var submit = this.$el.querySelector(".submit");
      submit.scrollIntoView({ behavior: "smooth" });
    },
    removeUserStyle() {
      this.userStyle = false;
      this.clothImage = null;
      this.userStyleSrc = "";
      this.submitDisable = true;
    },
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#wrapper h1 {
  margin: 1rem 0rem;
}

#wrapper h3 {
  margin-top: 0.2rem;
  margin-bottom: 0.8rem;
}

.container {
  display: flex;
}

.section {
  margin: 0.5rem 1rem;
  flex-grow: 1;
  width: 35%;
}

.section .options .vue-js-switch {
  margin: 0.5rem;
}

.image-container .image-flex {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  max-width: 400px;
  margin: 0 auto;
}

.image-container .image-item {
  width: 25%;
  margin: 3px;
  padding: 2px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.055);
  transition: all 0.2s ease-in-out;
}

.image-container .image-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  overflow: hidden;
}

img.selected {
  box-sizing: border-box;
  border: 4px solid #0088cc;
}

.result-container {
  min-width: 200px;
  min-height: 200px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #ffffff;
  padding: 0.5rem;
  display: flex;
  justify-content: center;
  align-items: center;
}

.result-container img {
  border: 1px solid #eee;
  border-radius: 0.2rem;
  width: 50%;
  height: 50%;
  object-fit: cover;
  overflow: hidden;
}

.hint {
  font-weight: 200;
  font-size: 1rem;
  color: #95a5a6;
}

.overlay {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  color: #bdc3c7;
}

.overlay .content {
  margin: 1rem;
}

@media only screen and (max-width: 800px) {
  .container {
    flex-direction: column;
  }

  .section {
    margin: 0;
    width: 100%;
  }

  .canvas-wrapper {
    max-height: 500px;
  }
}

.canvas-wrapper {
  max-height: 400px;
  height: auto;
  padding: 10px;
}

.btn {
  background-color: #008cba;
  border: none;
  border-radius: 0.3em;
  color: white;
  padding: 0.5em 1em;
  margin: 0.5em;
  font-size: 1rem;
  font-family: inherit;
  font-weight: 400;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  -webkit-transition-duration: 0.4s; /* Safari */
  transition-duration: 0.4s;
  cursor: pointer;
}

.btn:hover {
  background: #34495e;
}

.btn.disabled,
.btn[disabled],
fieldset[disabled] .btn {
  pointer-events: none;
  cursor: not-allowed;
  box-shadow: none;
  opacity: 0.5;
}

input[type="file"] {
  display: none;
}

.style-hint form {
  display: inline;
}

.style-hint {
  font-weight: 200;
  font-size: 0.8rem;
  color: #95a5a6;
}

.style-hint .upload-label {
  text-decoration: underline;
}

.upload-label {
  cursor: pointer;
  font-weight: 200;
  font-size: 0.8rem;
  color: #95a5a6;
}

.upload-style img {
  width: 192px;
  height: 256px;
  margin: 0.2rem;
  padding: 2px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.055);
  transition: all 0.2s ease-in-out;
}

.upload-style .remove {
  cursor: pointer;
  font-weight: 200;
  font-size: 0.8rem;
  color: #95a5a6;
  text-decoration: underline;
}

span.clear {
  background: url("../assets/icons/eraser.svg") no-repeat top left;
  background-size: contain;
  cursor: pointer;
  display: inline-block;
  height: 1rem;
  width: 1rem;
}

span.submit {
  background: url("../assets/icons/palette.svg") no-repeat top left;
  background-size: contain;
  cursor: pointer;
  display: inline-block;
  height: 1rem;
  width: 1rem;
}

span.upload {
  background: url("../assets/icons/upload.svg") no-repeat top left;
  background-size: contain;
  cursor: pointer;
  display: inline-block;
  height: 1rem;
  width: 1rem;
}

.half-circle-spinner,
.half-circle-spinner * {
  box-sizing: border-box;
}

.half-circle-spinner {
  width: 4rem;
  height: 4rem;
  border-radius: 100%;
  position: relative;
}

.half-circle-spinner .circle {
  content: "";
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 100%;
  border: calc(60px / 10) solid transparent;
}

.half-circle-spinner .circle.circle-1 {
  border-top-color: #bdc3c7;
  animation: half-circle-spinner-animation 1s infinite;
}

.half-circle-spinner .circle.circle-2 {
  border-bottom-color: #bdc3c7;
  animation: half-circle-spinner-animation 1s infinite alternate;
}

@keyframes half-circle-spinner-animation {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
