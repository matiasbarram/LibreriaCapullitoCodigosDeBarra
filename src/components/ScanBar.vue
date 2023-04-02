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

const doCamThings = async () => {
  /* get user's permission to muck around with video devices */
  const tempStream = await navigator.mediaDevices.getUserMedia({ video: true })
  const devices = await navigator.mediaDevices.enumerateDevices()
  let frontDeviceId
  let backDeviceId
  if (devices.length > 0) {
    /* defaults so all this will work on a desktop */
    frontDeviceId = devices[0].deviceId;
    backDeviceId = devices[0].deviceId;
  }
  /* look for front and back devices */
  devices.forEach(device => {
    if (device.kind === 'videoinput') {
      if (device.label && device.label.length > 0) {
        if (device.label.toLowerCase().indexOf('back') >= 0)
          backDeviceId = device.deviceId
        else if (device.label.toLowerCase().indexOf('front') >= 0)
          frontDeviceId = device.deviceId
      }
    }
  })
  /* close the temp stream */
  const tracks = tempStream.getTracks()
  if (tracks)
    for (let t = 0; t < tracks.length; t++) tracks[t].stop()
  /* open the device you want */
  const constraints = {
    video: true,
    deviceId: { exact: backDeviceId }
  }

  return navigator.mediaDevices.getUserMedia(constraints)
}

onMounted(() => {
  start()
  detecting()
  if ('mediaDevices' in navigator && 'getUserMedia' in navigator.mediaDevices) {
    navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } })
      .then(function (stream) {
        var video = document.querySelector("#videoWindow");
        video.srcObject = doCamThings();
      })
      .catch(function (error) {
        console.error("Error al acceder a la cámara: " + error);
      });
  }

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