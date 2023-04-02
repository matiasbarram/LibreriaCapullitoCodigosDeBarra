<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div id="videoWindow" class="video"></div>
    <select name="input-stream_constraints" id="deviceSelection">
    </select>
  </div>
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
  initCameraSelection()
  start()
  detecting()

})

// inputStream: {
//       name: "live",
//       type: "LiveStream",
//       target: document.querySelector("#videoWindow"),
//       constraints: {
//         width: 640,
//         height: 1281,
//         facingMode: "environment",

//       },
//     },

const start = async () => {
  const config = {
    locate: true,
    inputStream: {
      type: "LiveStream",
      target: document.querySelector("#videoWindow"),

      constraints: {
        width: { min: 640 },
        height: { min: 480 },
        facingMode: "environment",
        aspectRatio: { min: 1, max: 2 }
      }
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

const initCameraSelection = async () => {
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
</style>