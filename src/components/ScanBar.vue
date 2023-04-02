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

const constraints = {
  video: {
    facingMode: { ideal: 'environment' },
  }
};

onMounted(() => {
  start()
  detecting()

})


const start = async () => {
  const config = {
    locate: true,
    inputStream: {
      name: "live",
      type: "LiveStream",
      target: document.querySelector("#videoWindow"),
      constraints: {
        width: 640,
        height: 1281,
        facingMode: "environment",

      },
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

  const stream = await navigator.mediaDevices.getUserMedia(constraints)
  console.log(stream)
  const video = document.querySelector('video');
  video.srcObject = stream;
  video.play();
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