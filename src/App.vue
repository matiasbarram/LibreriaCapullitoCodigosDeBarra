<template>
  <AppHeader />
  <section class="main-section">
    <Modal v-if="showModal" :visible="showModal" @close="showModal = false" :codigo="barCodeData" />
    <ScanBar v-else class="scanner" @emitData="loadData" />
    <h1 class="main-section-title">Escanea tu producto para saber detalles</h1>
  </section>
  <button @click="getCameras">Test</button>
  <h2>{{ cameras }}</h2>
</template>

<script setup>
import AppHeader from './components/AppHeader.vue'
import ScanBar from './components/ScanBar.vue'
import Modal from './components/Modal.vue'
import { onMounted, ref, watch } from 'vue';

const showModal = ref(false);
const barCodeData = ref("")
const cameras = ref([])

const constraints = {
  video: {
    width: {
      min: 1280,
      ideal: 1920,
      max: 2560,
    },
    height: {
      min: 720,
      ideal: 1080,
      max: 1440
    },
  }
};

const getCameras = async () => {
  const raw = await navigator.mediaDevices.enumerateDevices()
  raw.forEach(device => {
    const temp = { "kind": device.kind, "label": device.label }
    cameras.value.push(temp)
  });

}

onMounted(() => {
})

const loadData = (scannedBarCode) => {
  showModal.value = true
  barCodeData.value = scannedBarCode
}
</script>

<style scoped>
.main-section {
  text-align: center;
  padding: 40px 20px;
}

.main-section-title {
  margin-top: 20px;
  font-size: 20px;
  font-weight: 700;
}

.scanner {
  margin: 20px 0;
}

.modal-box {
  display: flex;
  justify-content: center;
  justify-items: center;
}
</style>
