<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div id="videoWindow" class="video"></div>
    <select name="input-stream_constraints" id="deviceSelection" v-model="selectedCamera" @change="onChange()">
    </select>
    <div class="debug">{{ debug }}</div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Quagga from '@ericblade/quagga2'; // ES6

const emit = defineEmits(['emitData'])
const loading = ref(true)
const selectedCamera = ref("")
const debug = ref([])

const onChange = () => {
  console.log('The new value is: ', selectedCamera.value)
  Quagga.stop()
  start({
    width: { min: 640 },
    height: { min: 480 },
    facingMode: "environment",
    aspectRatio: { min: 1, max: 2 },
    deviceId: selectedCamera.value
  })
}

const selectDefaultCamera = () => {
  const activeStreamLabel = Quagga.CameraAccess.getActiveStreamLabel();
  console.log("Camara activa: " + activeStreamLabel)
  Quagga.CameraAccess.enumerateVideoDevices().then(function (devices) {
    devices.forEach(function (device) {
      console.log("Camera: " + device.label)
      debug.value.push({ "label": device.deviceId, "device": device.groupId })
      if (device.label === activeStreamLabel) {
        let defaultDeviceId = device.deviceId;
        selectedCamera.value = defaultDeviceId
        console.log("El deviceId por defecto es:", defaultDeviceId);
      }
    });
  })
}

onMounted(async () => {
  await initCameraSelector()
  await start({
    width: { min: 640 },
    height: { min: 480 },
    facingMode: "environment",
    aspectRatio: { min: 1, max: 2 }
  })
  detecting()

})



const start = async (constraints) => {
  const config = {
    locate: true,
    inputStream: {
      type: "LiveStream",
      target: document.querySelector("#videoWindow"),
      constraints: constraints
    },
    locator: {
      patchSize: "medium",
      halfSample: true
    },
    numOfWorkers: 2,
    frequency: 10,
    decoder: {
      readers: [{
        format: "code_128_reader",
        config: {}
      }]
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
    if (selectedCamera.value == "") {
      selectDefaultCamera()
    }

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

const initCameraSelector = async () => {
  var streamLabel = Quagga.CameraAccess.getActiveStreamLabel();

  return Quagga.CameraAccess.enumerateVideoDevices()
    .then(function (devices) {
      function pruneText(text) {
        return text.length > 30 ? text.substr(0, 30) : text;
      }
      var $deviceSelection = document.getElementById("deviceSelection");
      while ($deviceSelection.firstChild) {
        $deviceSelection.removeChild($deviceSelection.firstChild);
      }
      devices.forEach(function (device) {
        var $option = document.createElement("option");
        $option.value = device.deviceId || device.id;
        $option.appendChild(document.createTextNode(pruneText(device.label || device.deviceId || device.id)));
        $option.selected = streamLabel === device.label;
        $deviceSelection.appendChild($option);
      });
    });
}

</script>

<style>
canvas.drawingBuffer {
  display: none !important;
}

select {
  padding: 8px;
  font-size: 16px;
  border-radius: 4px;
  border: 1px solid #ccc;
  background-color: #fff;
  color: #333;
  margin: 20px 0;
}

.debug {
  overflow: scroll
}
</style>