<template>
    <div class="modal">
        <div class="modal-content" ref="target">
            <div class="close-action" @click="closeModal">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="close-icon">
                    <path fill="#000" fill-rule="evenodd"
                        d="M12 10.586L8.707 7.293a1 1 0 0 0-1.414 1.414L10.586 12l-3.293 3.293a1 1 0 1 0 1.414 1.414L12 13.414l3.293 3.293a1 1 0 1 0 1.414-1.414L13.414 12l3.293-3.293a1 1 0 0 0-1.414-1.414L12 10.586z" />
                </svg>
            </div>
            <template v-if="!dataFromApi">
                <h2>Estamos cargando la información</h2>
                <Loader />
            </template>

            <template v-else>
                <h2>Probando el modal </h2>
                <p class="product-info"> El codigo es: {{ dataFromApi }} </p>
            </template>
        </div>
    </div>
</template>
  
<script setup>
import { onMounted, ref, watch } from 'vue';
import Loader from './Loader.vue'
import { onClickOutside } from '@vueuse/core'

const target = ref(null)
const dataFromApi = ref(null)

const emit = defineEmits(['close'])
const props = defineProps({
    visible: Boolean,
    codigo: String,
    productData: Object
})

onMounted(() => {
    fetchDataFromApi()
})

onClickOutside(target, (event) => closeModal())


const fetchDataFromApi = async () => {
    try {
        const response = await fetch('https://reqres.in/api/users?page=2');
        const data = await response.json();
        dataFromApi.value = data;
    } catch (error) {
        console.error(error);
    }
}
const closeModal = () => {
    emit('close');
}

</script>
  
<style>
.product-info {
    overflow: scroll;
}

.close-icon {
    width: 35px;
    height: 35px;
}

.modal {
    position: fixed;
    z-index: 3;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-background {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
}

.modal-content {
    display: grid;
    height: 80%;
    width: 80%;
    background-color: #fff;
    padding: 20px;
    border-radius: 4px;
}

.close-action {
    justify-self: right;
}
</style>
  