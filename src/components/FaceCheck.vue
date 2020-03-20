<template>
  <div>
    <div class="parent hoge">
      <div class="v1">
        <img :src="imgsrc" ref="image" class="i2" />
      </div>
      <div class="v1">
        <canvas ref="canvas4reco" class="c2"></canvas>
      </div>
      <div v-if="loading" class="loader">Loading...</div>
    </div>
  </div>
</template>

<script>
import * as faceapi from "face-api.js";
export default {
  name: "FaceCheck",
  props: ["imgsrc"],
  data: function() {
    return {
      MODEL: "/models/weights",
      loading: false
    };
  },
  computed: {
    image() {
      return this.$refs["image"];
    },
    canvas4reco() {
      return this.$refs["canvas4reco"];
    }
  },
  methods: {
    getCheck() {
      // eslint-disable-next-line no-async-promise-executor
      return new Promise(async resolve => {
        const urls = [
          // please add your photos you want to check.
          // "/images/test1.png",
          // "/images/test2.png",
          // "/images/test3.png",
          // "/images/test4.png"
        ];
        let descriptors = [];
        for (let i = 0; i < urls.length; i++) {
          let url = urls[i];
          const image = await faceapi.fetchImage(url);
          const detection = await faceapi
            .detectSingleFace(image)
            .withFaceLandmarks()
            .withFaceDescriptor();
          descriptors.push(detection.descriptor);
        }
        resolve(new faceapi.LabeledFaceDescriptors("It is ME !", descriptors));
      });
    },
    clickReco() {
      this.loading = true;
      Promise.all([
        faceapi.nets.faceLandmark68Net.loadFromUri(this.MODEL),
        faceapi.nets.faceRecognitionNet.loadFromUri(this.MODEL),
        faceapi.nets.ssdMobilenetv1.loadFromUri(this.MODEL)
      ]).then(async () => {
        const size = {
          width: this.image.width,
          height: this.image.height
        };

        const detection = await faceapi
          .detectSingleFace(this.image)
          .withFaceLandmarks()
          .withFaceDescriptor();

        this.getCheck().then(Check => {
          const faceMatcher = new faceapi.FaceMatcher(Check, 0.6);
          const result = faceMatcher.findBestMatch(detection.descriptor);

          const resize = faceapi.resizeResults(detection, size);
          const box = resize.detection.box;
          const drawBox = new faceapi.draw.DrawBox(box, {
            label: result.toString()
          });
          faceapi.matchDimensions(this.canvas4reco, size);
          drawBox.draw(this.canvas4reco);

          this.loading = false;
        });
      });
    }
  },
  mounted() {}
};
</script>

<style scoped>
.check {
  padding: 30px 0;
  margin: 50px auto;
  color: white;
  background: #61c1be;
  box-shadow: 0px 0px 0px 10px #61c1be;
  border: dashed 2px white;
  border-radius: 100px;
  cursor: pointer;
  width: 100px;
  height: 40px;
  font-size: 20px;
}

.parent {
  position: relative;
}
.parent * {
  position: absolute;
}
.statics {
  padding: 400px 0;
  position: static;
}
.loader {
  color: white;
  font-size: 90px;
  text-indent: -9999em;
  overflow: hidden;
  width: 1em;
  height: 1em;
  border-radius: 50%;
  margin: 200px 350px;
  /* position: relative; */
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
  -webkit-animation: load6 1.7s infinite ease, round 1.7s infinite ease;
  animation: load6 1.7s infinite ease, round 1.7s infinite ease;
}
@-webkit-keyframes load6 {
  0% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em,
      0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  5%,
  95% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em,
      0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  10%,
  59% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.087em -0.825em 0 -0.42em,
      -0.173em -0.812em 0 -0.44em, -0.256em -0.789em 0 -0.46em,
      -0.297em -0.775em 0 -0.477em;
  }
  20% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.338em -0.758em 0 -0.42em,
      -0.555em -0.617em 0 -0.44em, -0.671em -0.488em 0 -0.46em,
      -0.749em -0.34em 0 -0.477em;
  }
  38% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.377em -0.74em 0 -0.42em,
      -0.645em -0.522em 0 -0.44em, -0.775em -0.297em 0 -0.46em,
      -0.82em -0.09em 0 -0.477em;
  }
  100% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em,
      0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
}
@keyframes load6 {
  0% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em,
      0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  5%,
  95% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em,
      0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  10%,
  59% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.087em -0.825em 0 -0.42em,
      -0.173em -0.812em 0 -0.44em, -0.256em -0.789em 0 -0.46em,
      -0.297em -0.775em 0 -0.477em;
  }
  20% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.338em -0.758em 0 -0.42em,
      -0.555em -0.617em 0 -0.44em, -0.671em -0.488em 0 -0.46em,
      -0.749em -0.34em 0 -0.477em;
  }
  38% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.377em -0.74em 0 -0.42em,
      -0.645em -0.522em 0 -0.44em, -0.775em -0.297em 0 -0.46em,
      -0.82em -0.09em 0 -0.477em;
  }
  100% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em,
      0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
}
@-webkit-keyframes round {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@keyframes round {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
</style>

