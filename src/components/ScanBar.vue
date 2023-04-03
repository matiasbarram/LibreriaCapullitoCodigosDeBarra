<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div id="videoWindow" class="video"></div>
    <div v-show="selectedCamera" style="margin:10px 0">
      <div class="alert alert-light" role="alert">
        Si tienes problemas intenta con otra cámara.
      </div>
      <select name="input-stream_constraints" id="deviceSelection" v-model="selectedCamera" @change="onChange()"
        @blur="isOpen = false" @keydown.enter="isOpen = false">
      </select>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Quagga from '@ericblade/quagga2';

const emit = defineEmits(['emitData'])
const loading = ref(true)
const selectedCamera = ref("")


const onChange = async () => {
  console.log('The new value is: ', selectedCamera.value)
  await Quagga.stop()
  loading.value = true
  await start({
    deviceId: selectedCamera.value
  })
  loading.value = false
}


function pruneText(text) {
  return text.length > 30 ? text.substr(0, 30) : text;
}

const initCameraSelector = (devices) => {
  var $deviceSelection = document.getElementById("deviceSelection");
  while ($deviceSelection.firstChild) {
    $deviceSelection.removeChild($deviceSelection.firstChild);
  }
  devices.forEach(function (device) {
    var $option = document.createElement("option");
    $option.value = device.deviceId || device.id;
    $option.appendChild(document.createTextNode(pruneText(device.label || device.deviceId || device.id)));
    $deviceSelection.appendChild($option);
  });
}

const DeviceDefaultCamera = async () => {
  var defaultDeviceId = null
  // Si es android seleccionamos la camara camera2 0 por defecto
  const devices = await Quagga.CameraAccess.enumerateVideoDevices();
  devices.forEach(function (device) {
    console.log("test getDefaultCamera: " + device.label)
    if (navigator.userAgent.match(/Android/i)) {
      if (device.label.includes("camera2 0")) {
        console.log("Cambiando camara por defecto" + device.deviceId)
        defaultDeviceId = device.deviceId
      }
    }
  });
  if (navigator.userAgent.match(/Android/i)) {
    initCameraSelector(devices)
  }
  return defaultDeviceId
}


onMounted(async () => {
  const defaultDeviceId = navigator.userAgent.match(/Android/i) ? await DeviceDefaultCamera() : null
  const constraints = defaultDeviceId ? { deviceId: defaultDeviceId } : {}
  await start(constraints)
  detecting()
})

const selectDefaultCamera = async () => {
  const activeStreamLabel = Quagga.CameraAccess.getActiveStreamLabel();
  console.log("Camara activa: " + activeStreamLabel)
  const devices = await Quagga.CameraAccess.enumerateVideoDevices()
  devices.forEach(function (device) {
    if (device.label === activeStreamLabel) {
      let defaultDeviceId = device.deviceId;
      selectedCamera.value = defaultDeviceId
      console.log("El deviceId por defecto es:", defaultDeviceId);
    }
  });
}

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

  await Quagga.init(config, err => {
    if (err) {
      console.log(err);
      return;
    }
    console.log("initialization complete");
    loading.value = false
    Quagga.start()
    if (navigator.userAgent.match(/Android/i)) {
      selectDefaultCamera()
    }


  })
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

select {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  width: 100%;
  height: 100%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  background-color: #fff;
  cursor: pointer;
}


.alert {
  position: relative;
  text-align: left;
  padding: 0.75rem 1.25rem;
  margin-bottom: 1rem;
  border: 1px solid transparent;
  border-radius: 0.25rem;
}

.alert-light {
  color: #0c5460;
  background-color: #d1ecf1;
  border-color: #bee5eb;
}
</style>