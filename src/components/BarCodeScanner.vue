<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div v-show="isAndroid" style="margin:10px 0">
      <Alert message="¿Problemas al escanear?" subMessage="Intenta con otra cámara." />
    </div>
    <div id="videoWindow" class="video"></div>
    <div v-show="isAndroid" style="margin:10px 0">
      <div class="options-container">
        <div v-show="hasTorchCap">
          <button class="torch" @click="onChangeTorchTest()">
            <svg xmlns="http://www.w3.org/2000/svg" width="60" height="60" fill="currentColor" class="bi bi-lightning"
              viewBox="0 0 16 16">
              <path v-if="torchOn"
                d="M5.52.359A.5.5 0 0 1 6 0h4a.5.5 0 0 1 .474.658L8.694 6H12.5a.5.5 0 0 1 .395.807l-7 9a.5.5 0 0 1-.873-.454L6.823 9.5H3.5a.5.5 0 0 1-.48-.641l2.5-8.5zM6.374 1 4.168 8.5H7.5a.5.5 0 0 1 .478.647L6.78 13.04 11.478 7H8a.5.5 0 0 1-.474-.658L9.306 1H6.374z" />
              <path v-else
                d="M5.52.359A.5.5 0 0 1 6 0h4a.5.5 0 0 1 .474.658L8.694 6H12.5a.5.5 0 0 1 .395.807l-7 9a.5.5 0 0 1-.873-.454L6.823 9.5H3.5a.5.5 0 0 1-.48-.641l2.5-8.5z" />
            </svg>
          </button>
        </div>
        <div v-show="hasZoomCap" class="zoom-buttons-container">
          <button class="zoom" @click="onChangeZoomTest('down')" :disabled="actualZoomValue <= zoomValue.min">-</button>
          <button class="zoom" @click="onChangeZoomTest('up')" :disabled="actualZoomValue >= zoomValue.max">+</button>
        </div>
        <div class="select-container">
          <select v-model="selectedCamera" @change="onChange()">
            <option v-for="option in options" :value="option.value">{{ option.label }}</option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Alert from "./Alert.vue"
import Quagga from '@ericblade/quagga2';

const emit = defineEmits(['emitData'])
const loading = ref(true)
const isAndroid = ref(false)
const hasZoomCap = ref(false)
const hasTorchCap = ref(false)
const torchOn = ref(false)
const actualZoomValue = ref(0)
const zoomValue = ref({})
const selectedCamera = ref("")
const options = ref([])

const checkAndroidCamera20 = async () => {
  const deviceId = localStorage.getItem("deviceId")
  if (deviceId) {
    console.log("deviceId", deviceId)
    return
  }
  const defaultDeviceLabel = Quagga.CameraAccess.getActiveStreamLabel()
  console.log("defaultDevice", defaultDeviceLabel)
  if (defaultDeviceLabel.includes("camera2 0")) {
    return
  }

  const devices = await Quagga.CameraAccess.enumerateVideoDevices()
  const camera20 = devices.find((device) => device.label.includes("camera2 0"))
  if (camera20) {
    console.log("camera20", camera20.label)
    selectedCamera.value = camera20.deviceId
    localStorage.setItem("deviceId", camera20.deviceId)
    await Quagga.stop();
    start({
      deviceId: camera20.deviceId
    })
  }
}


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

const onChangeZoomTest = (direction) => {
  if (direction == "up" && actualZoomValue.value < zoomValue.value.max) {
    actualZoomValue.value = actualZoomValue.value + zoomValue.value.step


  } else if (direction == "down" && actualZoomValue.value > zoomValue.value.min) {
    actualZoomValue.value = actualZoomValue.value - zoomValue.value.step
  }
  changeCapabilities({
    advanced: [{
      zoom: actualZoomValue.value
    }]
  })
  actualZoomValue.value = parseFloat(actualZoomValue.value.toFixed(1))
  console.log("type" + typeof actualZoomValue.value)
  console.log("Zoom value" + actualZoomValue.value)
}

const onChangeTorchTest = () => {
  torchOn.value = !torchOn.value
  console.log("Torch value" + torchOn.value)
  changeCapabilities({
    advanced: [{
      torch: torchOn.value
    }]
  })
}

const changeCapabilities = async (props) => {
  const track = Quagga.CameraAccess.getActiveTrack();
  await track.applyConstraints(props);
}


onMounted(() => {
  start({
    deviceId: localStorage.getItem("deviceId")
  })
})


const createConstraints = (constraints) => {
  if (navigator.userAgent.match(/iPhone/i)) {
    console.log("iPhone")
    const data = {
      inputStream: {
        name: "Live",
        type: "LiveStream",
        facingMode: "environment",
        constraints: {
          video: {
            aspectRatio: {
              ideal: 1920 / 1080,
            }
          }
        },
        target: document.querySelector('#videoWindow')
      },
      locator: {
        halfSample: true,
        patchSize: "small",
        debug: {
          showCanvas: false,
          showPatches: false,
          showFoundPatches: false,
          showSkeleton: false,
          showLabels: false,
          showPatchLabels: false,
          showRemainingPatchLabels: false,
          boxFromPatches: {
            showTransformed: false,
            showTransformedBox: false,
            showBB: false
          }
        }
      },
      locate: true,
      numberOfWorkers: navigator.hardwareConcurrency,
      decoder: {
        readers: ["ean_reader"]
      }
    }
    console.log(JSON.stringify(data, null, 2));
    return data
  }
  else {
    console.log("Android or Other")
    return {
      locate: true,
      inputStream: {
        type: "LiveStream",
        target: document.querySelector("#videoWindow"),
        constraints: {
          aspectRatio: {
            ideal: 1,
          },
          ...constraints
        }
      },
      locator: {
        patchSize: "large",
        halfSample: true
      },
      numOfWorkers: navigator.hardwareConcurrency,
      frequency: 10,
      decoder: {
        readers: ["ean_reader"],
        multiple: false
      },
    };
  }

}

const start = (constraints) => {
  const config = createConstraints(constraints)
  console.log("Starting", config)
  Quagga.init(config, (err) => {
    if (err) {
      console.log(err);
      return
    }
    console.log("Initialization finished. Ready to start");
    Quagga.start();
    loading.value = false
    detecting()
    checkZoomCapability()
    checkTorchCapability()
    if (selectedCamera.value == "") {
      if (navigator.userAgent.match(/Android/i) || navigator.userAgent.match(/Windows/i)) {
        isAndroid.value = true
        checkAndroidCamera20().then(() => {
          addSelectOptions()
          selectDefaultCamera()
        })
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

const getMedian = (arr) => {
  arr.sort((a, b) => a - b);
  const half = Math.floor(arr.length / 2);
  if (arr.length % 2 === 1) // Odd length
    return arr[half];
  return (arr[half - 1] + arr[half]) / 2.0;
}

const detecting = () => {
  Quagga.onDetected((data) => {
    const errors = data.codeResult.decodedCodes
      .filter(_ => _.error !== undefined)
      .map(_ => _.error);
    const err = getMedian(errors);

    console.log("Total error: " + err)
    if (err < 0.1) {
      const barcode = data.codeResult.code
      Quagga.stop();
      console.log("Found: ", data)
      emit("emitData", barcode);
    } else {
      console.log("Error: ", err)
    }

  });
}

const checkZoomCapability = () => {
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
  }
  actualZoomValue.value = capabilities.zoom.min
  console.log("Zoom capabilities: ", capabilities.zoom);
  console.log(JSON.stringify(capabilities, null, 2));
}

const checkTorchCapability = () => {
  var track = Quagga.CameraAccess.getActiveTrack();
  var capabilities = {};
  if (typeof track.getCapabilities === 'function') {
    capabilities = track.getCapabilities();
  }
  if (!('torch' in capabilities)) {
    return Promise.reject('Torch is not supported by ' + track.label);
  }
  hasTorchCap.value = true
  console.log("torch capabilities: ", capabilities.torch);
}

</script>

<style>
/* canvas.drawingBuffer {
  display: none !important;
} */

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

.options-container {
  display: flex;
  grid-column-gap: 10px;
  margin: 10px 0;
  justify-content: space-between;

}

.zoom-buttons-container {
  display: flex;
  column-gap: 20px;
}

.zoom {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  background-color: #0c5460;
  color: #fff;
  font-size: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.zoom:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.bi-lightning {
  color: #ffc107;
}
</style>