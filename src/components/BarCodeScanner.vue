<template>
  <Loader v-if="loading" />
  <div v-show="!loading">
    <div id="videoWindow" class="video"></div>
    <div v-show="selectedCamera" style="margin:10px 0">
      <div class="alert alert-light" role="alert">
        <span style="font-weight: 600;">¿Problemas al escanear?</span>
        Intenta con otra cámara.
      </div>
      <select name="input-stream_constraints" id="deviceSelection" v-model="selectedCamera" @change="onChange()"
        @blur="isOpen = false" @keydown.enter="isOpen = false">
      </select>
    </div>
  </div>
</template>

<script setup>
import { onBeforeMount, onMounted, ref } from "vue";
import Loader from "./Loader.vue"
import Quagga from '@ericblade/quagga2';

const emit = defineEmits(['emitData'])
const loading = ref(true)
const selectedCamera = ref("")


const enumerateDevices = async () => {
  try {
    const devices = await navigator.mediaDevices.enumerateDevices();
    const cameras = devices.filter(device => device.kind === 'videoinput');
    console.log('IDs de cámaras disponibles:', cameras.map(camera => camera.deviceId));
    return
  } catch (error) {
    console.error('Error al enumerar dispositivos', error);
  }
};

onMounted(() => {
  start()
})


const start = () => {
  Quagga.init({
    inputStream: {
      name: "Live",
      type: "LiveStream",
      target: document.querySelector('#videoWindow'),
      constraints: {
        focusMode: "manual",
        width: 640,
        height: 480,
        facingMode: "environment",
      },
    },
    decoder: {
      readers: ["code_128_reader", "ean_reader", "ean_8_reader", "code_39_reader", "code_39_vin_reader", "codabar_reader", "upc_reader", "upc_e_reader", "i2of5_reader"]
    },
  }, (err) => {
    if (err) {
      console.log(err);
      return
    }
    console.log("Initialization finished. Ready to start");
    Quagga.start();
    loading.value = false
    detecting()

  });
}

const selector = () => {
  Quagga.onProcessed((result) => {
    var drawingCtx = Quagga.canvas.ctx.overlay,
      drawingCanvas = Quagga.canvas.dom.overlay;

    if (result) {
      if (result.boxes) {
        drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
        result.boxes.filter(function (box) {
          return box !== result.box;
        }).forEach(function (box) {
          Quagga.ImageDebug.drawPath(box, {
            x: 0,
            y: 1
          }, drawingCtx, {
            color: "green",
            lineWidth: 2
          });
        });
      }

      if (result.box) {
        Quagga.ImageDebug.drawPath(result.box, {
          x: 0,
          y: 1
        }, drawingCtx, {
          color: "#00F",
          lineWidth: 2
        });
      }

      if (result.codeResult && result.codeResult.code) {
        Quagga.ImageDebug.drawPath(result.line, {
          x: 'x',
          y: 'y'
        }, drawingCtx, {
          color: 'red',
          lineWidth: 3
        });
      }
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