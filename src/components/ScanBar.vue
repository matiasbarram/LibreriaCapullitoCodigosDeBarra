<template>
  <Loader v-if="loading" />
  <StreamBarcodeReader v-show="!loading" @init="OnInit" @decode="onDecode" @loaded="onLoaded">
    <Loader v-if="loading" />
  </StreamBarcodeReader>
</template>

<script setup>
import { ref } from "vue";
import { StreamBarcodeReader } from "vue-barcode-reader";
import Loader from "./Loader.vue"

const emit = defineEmits(['emitData'])
const loading = ref(true)

const onDecode = (barcode) => {
  console.log(barcode)
  emit("emitData", barcode);
}


const onLoaded = () => {
  loading.value = false
  console.log("Ready to scan!")

}
</script>