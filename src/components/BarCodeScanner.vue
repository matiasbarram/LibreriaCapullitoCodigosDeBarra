<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div v-show="isAndroid" style="margin:10px 0">
      <select v-model="selectedCamera" @change="onChange()">
        <option v-for="option in options" :value="option.value">{{ option.label }}</option>
      </select>
    </div>
    <div id="videoWindow" class="video"></div>
    <div v-show="isAndroid" style="margin:10px 0">
      <input v-if="hasZoomCap" type="range" v-model="zoomValue" :min="zoomValue.min" :max="zoomValue.max"
        @change="onChangeZoom()" :step="zoomValue.step">
      <div class="alert alert-light" role="alert">
        <span style="font-weight: 600;">¿Problemas al escanear?</span>
        Intenta con otra cámara.
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Quagga from '@ericblade/quagga2';

const emit = defineEmits(['emitData'])
const loading = ref(true)
const isAndroid = ref(false)
const hasZoomCap = ref(true)
const zoomValue = ref({})
const selectedCamera = ref("")
const options = ref([])


const onChange = async () => {
  const deviceId = selectedCamera.value
  const constraints = {
    deviceId: {
      exact: deviceId
    }
  }
  localStorage.setItem("deviceId", deviceId)
  await Quagga.stop();
  start(constraints)
}

const onChangeZoom = async () => {
  const track = Quagga.CameraAccess.getActiveTrack();
  await track.applyConstraints({
    advanced: [{
      zoom: zoomValue.value
    }]
  });
}


onMounted(() => {
  start({
    deviceId: localStorage.getItem("deviceId")
  })
})


const start = (constraints) => {
  const config = {
    locate: true,
    inputStream: {
      type: "LiveStream",
      target: document.querySelector("#videoWindow"),
      constraints: constraints
    },
    locator: {
      patchSize: "small",
      halfSample: true
    },
    numOfWorkers: navigator.hardwareConcurrency,
    frequency: 10,
    decoder: {
      readers: ["ean_reader", "ean_8_reader", "code_128_reader", "code_39_reader", "code_39_vin_reader", "codabar_reader", "upc_reader", "upc_e_reader", "i2of5_reader"],
      multiple: true
    },
  };
  Quagga.init(config, (err) => {
    if (err) {
      console.log(err);
      return
    }
    console.log("Initialization finished. Ready to start");
    Quagga.start();
    loading.value = false
    detecting()
    checkCapabilities() // test
    if (selectedCamera.value == "") {
      if (navigator.userAgent.match(/Android/i) || navigator.userAgent.match(/Windows/i)) {
        isAndroid.value = true
        addSelectOptions()
        selectDefaultCamera()
      }
    }
  })
}

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


const addSelectOptions = () => {
  navigator.mediaDevices.enumerateDevices().then((devices) => {
    devices.forEach((device) => {
      if (device.kind === 'videoinput') {
        const option = {}
        option.value = device.deviceId;
        option.label = device.label;
        console.log(option)
        options.value.push(option)
      }
    });
    selectedCamera.value = Quagga.CameraAccess.getActiveStreamLabel()
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


const checkCapabilities = () => {
  var track = Quagga.CameraAccess.getActiveTrack();
  var capabilities = {};
  if (typeof track.getCapabilities === 'function') {
    capabilities = track.getCapabilities();
  }
  if (!('zoom' in capabilities)) {
    return Promise.reject('Zoom is not supported by ' + track.label);
  }
  hasZoomCap.value = true
  zoomValue.value = {
    min: capabilities.zoom.min,
    max: capabilities.zoom.max,
    step: capabilities.zoom.step,
    value: track.getSettings().zoom
  }
  console.log("Zoom capabilities: ", capabilities.zoom);
  console.log(JSON.stringify(capabilities, null, 2));
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

input[type="range"] {
  margin: 18px 0;
  -webkit-appearance: none;
  margin-right: 15px;
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 5px;
  background-image: linear-gradient(#4e939f, #4e939f);
  background-size: 100%;
  background-repeat: no-repeat;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 20px;
  width: 20px;
  border-radius: 50%;
  background: #0c5460;
  cursor: ew-resize;
  box-shadow: 0 0 2px 0 #555;
  transition: background .3s ease-in-out;
}
</style>