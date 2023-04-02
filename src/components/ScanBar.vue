<template>
  <Loader v-if="loading" />
  <div v-show="!loading" id="videoWindow" class="video"></div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Quagga from '@ericblade/quagga2'; // ES6

const emit = defineEmits(['emitData'])
const loading = ref(true)
const videoConstraints = {
  video: {
    width: { ideal: 1280 },
    height: { ideal: 720 },
    facingMode: 'user',
    advanced: [
      {
        zoom: 1
      }
    ]
  }
};


onMounted(() => {
  start()
  detecting()

  navigator.mediaDevices.getUserMedia(videoConstraints)
    .then(function (mediaStream) {
      const video = document.querySelector('video');
      video.srcObject = mediaStream;
      video.onloadedmetadata = function (e) {
        video.play();
      };
    })
    .catch(function (err) { console.log(err.name + ": " + err.message); });

})


const start = () => {
  const config = {
    locate: true,
    inputStream: {
      name: "live",
      type: "LiveStream",
      target: document.querySelector("#videoWindow")
    },
    decoder: {
      readers: ["ean_reader"],
      multiple: true
    },
    locator: {
      halfSample: true,
      patchSize: "medium"
    }
  };

  Quagga.init(config, err => {
    if (err) {
      console.log(err);
      return;
    }
    console.log("initialization complete");
    loading.value = false
    Quagga.start();
  });
}

const detecting = () => {
  Quagga.onDetected((data) => {
    console.log(data);
    const foundResult = data[0];
    const barcode = foundResult.codeResult.code
    Quagga.stop();
    console.log("Found: " + barcode)
    emit("emitData", barcode);
  });

}

</script>

<style>
canvas.drawingBuffer {
  display: none !important;
}
</style>