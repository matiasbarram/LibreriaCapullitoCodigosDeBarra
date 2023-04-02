<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div id="videoWindow" class="video"></div>
    <div class="select-wrapper" :class="{ open: isOpen, selected: selectedValue }">
      <select name="input-stream_constraints" id="deviceSelection" v-model="selectedCamera" @change="onChange()"
        @blur="isOpen = false" @keydown.enter="isOpen = false">
      </select>
      <svg class="arrow" viewBox="0 0 24 24">
        <path d="M7.41,8.59L12,13.17l4.59-4.58L18,10l-6,6l-6-6L7.41,8.59z" />
      </svg>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Quagga from '@ericblade/quagga2';
import { computed } from "@vue/reactivity";

const emit = defineEmits(['emitData'])
const loading = ref(true)
const selectedCamera = ref("")
const debug = ref([])

const arrowPath = computed(() =>
  isOpen.value ? 'M7.41,8.59L12,13.17l4.59-4.58L18,10l-6,6l-6-6L7.41,8.59z' : 'M7.41,15.41L12,10.83l4.59,4.58L18,15l-6-6l-6,6L7.41,15.41z'
);

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
    debug.value.push({ "label": device.label, "id": device.deviceId })
    var $option = document.createElement("option");
    $option.value = device.deviceId || device.id;
    $option.appendChild(document.createTextNode(pruneText(device.label || device.deviceId || device.id)));
    $deviceSelection.appendChild($option);
  });
}

const AndroidDefaultCamera = async () => {
  var defaultDeviceId = null
  // Si es android seleccionamos la camara camera2 0 por defecto
  const devices = await Quagga.CameraAccess.enumerateVideoDevices();
  devices.forEach(function (device) {
    console.log("test getDefaultCamera: " + device.label)
    if (device.label.includes("camera2 0")) {
      console.log("Cambiando camara por defecto" + device.deviceId)
      defaultDeviceId = device.deviceId
    }
  });
  initCameraSelector(devices)
  return defaultDeviceId
}


onMounted(async () => {
  const defaultDeviceId = await AndroidDefaultCamera()
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
    selectDefaultCamera()


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

.debug {
  overflow: scroll
}

.select-wrapper {
  position: relative;
  width: 200px;
  height: 40px;
  margin: 20px;
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

.arrow {
  position: absolute;
  top: 50%;
  right: 10px;
  transform: translateY(-50%);
  width: 16px;
  height: 16px;
  fill: #999;
  transition: transform 0.3s ease-in-out;
  cursor: pointer;
}

.select-wrapper.open .arrow {
  transform: translateY(-50%) rotate(180deg);
}
</style>