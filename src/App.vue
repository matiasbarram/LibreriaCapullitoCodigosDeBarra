<template>
  <AppHeader />
  <section class="main-section">
    <Modal v-if="showModal" :visible="showModal" @close="showModal = false" :codigo="barCodeData" />
    <BarCodeScanner v-else class="scanner" @emitData="loadData" />
  </section>
</template>

<script setup>
import AppHeader from './components/AppHeader.vue'
import BarCodeScanner from './components/BarCodeScanner.vue'
import Modal from './components/Modal.vue'
import { onMounted, ref, watch } from 'vue';

const showModal = ref(false);
const barCodeData = ref("")
const cameras = ref([])


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
