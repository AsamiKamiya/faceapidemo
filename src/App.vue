<template>
  <div class="bg">
    <div id="app">
      <div class="title">Face API DEMO !</div>
      <div v-if="notcapture" class="parent">
        <div class="v1">
          <video ref="video" width="700" height="525" v-on:play="onPlay"></video>
        </div>
        <div class="c1">
          <canvas ref="canvas" width="700" height="525"></canvas>
        </div>
        <div v-if="isChecked">
          <div class="c1" v-if="emotion ==='happy'">
            <img src="@/assets/happy.png" class="emotion rightside fast" />
            <img src="@/assets/happy.png" class="emotion leftside fast" />
          </div>
          <div class="c1" v-if="emotion ==='sad'">
            <img src="@/assets/sad.png" class="emotion rightside slow" />
            <img src="@/assets/sad.png" class="emotion leftside slow" />
          </div>
          <div class="c1" v-if="emotion ==='angry'">
            <img src="@/assets/angry.png" class="emotion rightside fast expand" />
          </div>
          <div class="c1" v-if="emotion ==='surprised'">
            <div class="middle">
              <p class="emotionfont">!?</p>
            </div>
          </div>
          <div class="c1" v-if="emotion ==='neutral'"></div>
        </div>
      </div>
      <div v-else>
        <FaceCheck :imgsrc="caputuredata" ref="facecheck"></FaceCheck>
      </div>
      <canvas ref="caputure" width="700" height="525" hidden></canvas>
      <div class="footer">
        <img
          v-if="!isTracked"
          src="@/assets/faceb.png"
          class="buttonsL"
          v-on:click="changeTracking"
        />
        <img v-if="isTracked" src="@/assets/facea.png" class="buttonsL" v-on:click="changeTracking" />
        <img v-if="!isChecked" src="@/assets/emotb.png" class="buttonsL" v-on:click="changeEmotion" />
        <img v-if="isChecked" src="@/assets/emota.png" class="buttonsL" v-on:click="changeEmotion" />
        <div v-if="notcapture" v-on:click="saveCapture" class="capture">Capture</div>
        <div v-else v-on:click="checkFace" class="capture">Check</div>
        <img v-if="!isList" src="@/assets/logs.png" class="buttonsR" v-on:click="checkLists" />
        <img v-if="isList" src="@/assets/logsa.png" class="buttonsR" v-on:click="checkLists" />
        <img src="@/assets/reload.png" class="buttonsR" v-on:click="allClear" />
        <div v-if="isList" class="divphoto">
          <img v-for="(photo,index) in this.photos" :key="index" :src="photo" class="photolist" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as faceapi from "face-api.js";
import FaceCheck from "./components/FaceCheck";

export default {
  name: "App",
  components: {
    FaceCheck
  },
  el: "#app",
  data: function() {
    return {
      MODELS: "/models/weights/",
      notcapture: true,
      capturedata: "",
      emotion: "neutral",
      isChecked: false,
      isTracked: false,
      photos: [
        // please add your photos you want to check.
        // "/images/test1.png",
        // "/images/test2.png",
        // "/images/test3.png",
        // "/images/test4.png"
      ],
      isList: false
    };
  },
  watch: {
    toggle: function(newVal) {
      this.isemotion = newVal;
    }
  },
  computed: {
    video: function() {
      return this.$refs["video"];
    },
    canvas: function() {
      return this.$refs["canvas"];
    },
    caputure: function() {
      return this.$refs["caputure"];
    }
  },
  methods: {
    checkFace: function() {
      this.$refs.facecheck.clickReco();
    },
    changeEmotion: function() {
      if (!this.isChecked) {
        this.isChecked = true;
      } else {
        this.isChecked = false;
      }
    },
    changeTracking: function() {
      if (!this.isTracked) {
        this.isTracked = true;
        this.faceUrl = "facea";
      } else {
        this.isTracked = false;
        this.faceUrl = "faceb";
      }
    },
    checkLists: function() {
      if (!this.isList) {
        this.isList = true;
      } else {
        this.isList = false;
      }
    },
    allClear: function() {
      // clear function.
    },
    onPlay() {
      const video = this.video;
      const canvas = this.canvas;
      setInterval(async () => {
        // get info from camera.
        const detections = await faceapi
          .detectAllFaces(this.video, new faceapi.TinyFaceDetectorOptions())
          .withFaceLandmarks()
          .withFaceExpressions();

        const resize = faceapi.resizeResults(detections, {
          width: video.width,
          height: video.height
        });

        canvas.getContext("2d").clearRect(0, 0, canvas.width, canvas.height);
        if (this.isTracked) {
          faceapi.draw.drawDetections(canvas, resize);
          faceapi.draw.drawFaceLandmarks(canvas, resize);
          faceapi.draw.drawFaceExpressions(canvas, resize);
        }
        const expressionsObj = detections[0].expressions;
        let exp = [];
        let res = [];
        for (let key in expressionsObj) {
          exp.push(key);
          if (isNaN(Math.round(expressionsObj[key] * 100))) {
            res.push(0);
          } else {
            res.push(Math.round(expressionsObj[key] * 100));
          }
        }
        const index = res.indexOf(Math.max(...res));
        this.emotion = exp[index];
      }, 500);
    },
    saveCapture() {
      this.notcapture = false;
      this.caputure.getContext("2d").drawImage(this.video, 0, 0, 700, 525);
      this.caputuredata = this.caputure.toDataURL();
    }
  },
  mounted() {
    Promise.all([
      faceapi.nets.tinyFaceDetector.loadFromUri(this.MODELS),
      faceapi.nets.faceLandmark68Net.loadFromUri(this.MODELS),
      faceapi.nets.faceExpressionNet.loadFromUri(this.MODELS)
    ]).then(() => {
      navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
        this.video.srcObject = stream;
        this.video.play();
      });
    });
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Changa&display=swap");
body {
  background: radial-gradient(#61c1be 3px, transparent 4px),
    radial-gradient(#61c1be 3px, transparent 4px),
    linear-gradient(#fff 4px, transparent 0),
    linear-gradient(
      45deg,
      transparent 74px,
      transparent 75px,
      #84ccc9 75px,
      #84ccc9 76px,
      transparent 77px,
      transparent 109px
    ),
    linear-gradient(
      -45deg,
      transparent 75px,
      transparent 76px,
      #84ccc9 76px,
      #84ccc9 77px,
      transparent 78px,
      transparent 109px
    ),
    #fff;
  background-size: 109px 109px, 109px 109px, 100% 6px, 109px 109px, 109px 109px;
  background-position: 54px 55px, 0px 0px, 0px 0px, 0px 0px, 0px 0px;
  font-family: "Changa", sans-serif;
}
#app {
  /* font-family: Avenir, Helvetica, Arial, sans-serif; */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
  margin: 0 auto;
  width: 700px;
}
.title {
  margin: 50px auto 0px auto;
  color: white;
  background: #61c1be;
  cursor: pointer;
  width: 700px;
  height: 60px;
  font-size: 30px;
}
.footer {
  background: #61c1be;
  height: 130px;
  margin: 500px auto 0 auto;
  position: relative;
}
.footer * {
  display: inline-block;
  vertical-align: top;
  cursor: pointer;
}
.capture {
  padding: 30px 0;
  margin: 0px auto 0px auto;
  color: white;
  background: #61c1be;
  box-shadow: 0px 0px 0px 10px white;
  border: solid 2px white;
  border-radius: 100px;
  cursor: pointer;
  width: 100px;
  height: 40px;
  font-size: 20px;
}
.buttonsL {
  margin: 25px 50px 0 0;
  width: 50px;
  height: 50px;
}
.buttonsR {
  margin: 25px 0 0 50px;
  width: 50px;
  height: 50px;
}
.parent {
  position: relative;
}
.parent * {
  position: absolute;
}
.back {
  width: 510px;
  height: 410px;
  background-color: #2c3e50;
}
button {
  position: absolute;
}
.clear {
  position: static;
}
.emotion {
  width: 100px;
  height: 100px;
}
.fast {
  animation: flash 0.3s linear infinite;
}
.rightside {
  margin: 0 0 0 600px;
}
.leftside {
  margin: 400px 0 0 0;
}
.middle {
  margin: 0 0 0 500px;
}
.slow {
  animation: move 1s linear infinite;
}
.expand {
  animation: wide 0.3s linear infinite;
}
.emotionfont {
  font-size: 150px;
  color: white;
  text-shadow: -1px -1px 0 black, 1px -1px 0 black, -1px 1px 0 black,
    1px 1px 0 black;
}
@keyframes flash {
  0%,
  100% {
    opacity: 1;
  }

  50% {
    opacity: 0;
  }
}
@keyframes move {
  0% {
    opacity: 1;
    top: 0px;
  }

  100% {
    opacity: 1;
    top: 50px;
  }
}
@keyframes wide {
  0% {
    opacity: 1;
    transform: scale(1);
  }

  100% {
    opacity: 1;
    transform: scale(1.5);
  }
}

.switch input {
  display: none;
}

.switch label {
  padding: 12px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}
.switch label:before {
  padding: 6px 10px;
  content: "O N";
  border-radius: 6px 0 0 6px;
  background: silver;
  box-shadow: 0px 0px 3px 0px rgba(0, 0, 0, 0.1) inset;
  color: white;
}
.switch label:after {
  padding: 6px 10px;
  content: "OFF";
  border-radius: 0 6px 6px 0;
  background: #fff9b1;
  box-shadow: 0px 0px 3px 0px rgba(0, 0, 0, 0.1) inset;
  color: gray;
}
.switch input + label:hover:before {
  opacity: 0.5;
}
.switch input:checked + label:before {
  background: #61c1be;
  color: #fff;
  opacity: 1;
}
.switch input:checked + label:after {
  background: silver;
  color: #333;
}
.switch input:checked + label:hover:after {
  opacity: 0.5;
}

.photolist {
  width: 100px;
  height: 100px;
  margin: 10px;
}

.divphoto {
  position: relative;
  display: inline-block;
  margin: 1.5em 0;
  padding: 7px 10px;
  min-width: 120px;
  max-width: 100%;
  color: #555;
  font-size: 16px;
  background: #d4ecea;
}

.divphoto:before {
  content: "";
  position: absolute;
  top: -30px;
  left: 50%;
  margin-left: -15px;
  border: 15px solid transparent;
  border-bottom: 15px solid #d4ecea;
}
</style>
