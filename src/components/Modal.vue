<template>
    <div class="modal">
        <div class="modal-content">
            <div class="close-action" @click="closeModal">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="close-icon">
                    <path fill="#000" fill-rule="evenodd"
                        d="M12 10.586L8.707 7.293a1 1 0 0 0-1.414 1.414L10.586 12l-3.293 3.293a1 1 0 1 0 1.414 1.414L12 13.414l3.293 3.293a1 1 0 1 0 1.414-1.414L13.414 12l3.293-3.293a1 1 0 0 0-1.414-1.414L12 10.586z" />
                </svg>
            </div>

            <template v-if="isLoading">
                <h2>Estamos cargando la información de tu producto</h2>
                <Loader />
            </template>

            <template v-else-if="!isLoading && hasError" class="product_container">
                <h2>{{ errorMessage.message }}</h2>
                <p>{{ errorMessage.hint }}</p>
            </template>

            <template v-else>
                <div class="product_container">
                    <img class="product_image" :src="`https://www.libreriacapullito.cl/${apiRespose.imagen}`"
                        :alt="apiRespose.nombre">
                    <h2 class="product_name">{{ apiRespose.nombre }}</h2>
                    <p>{{ apiRespose.marca }}</p>
                </div>
                <div class="price_container">
                    <div class="product_price_container">
                        <p class="product_price">{{ formatPrice(apiRespose.precioNeto) }}</p><span>IVA incluido.</span>
                    </div>
                    <p v-if="apiRespose.precioOferta != 0" class="product_oferta">{{ apiRespose.precioOferta }}</p>
                </div>
                <small class="product_code">Codigo de tienda: {{ apiRespose.codigoTienda }}</small>
            </template>
        </div>
    </div>
</template>
  
<script setup>
import { onMounted, ref } from 'vue';
import Loader from './Loader.vue'

const apiRespose = ref({})
const isLoading = ref(true)
const hasError = ref(false)
const errorMessage = ref({})

const emit = defineEmits(['close'])
const props = defineProps({
    visible: Boolean,
    codigo: String,
})

onMounted(async () => {
    await fetchDataFromApi()
})


const formatPrice = (price) => {
    return new Intl.NumberFormat("es-CL", {
        style: "currency",
        currency: "CLP",
    }).format(price);
};

const fetchDataFromApi = async () => {
    try {
        console.log(props.codigo)
        const response = await fetch(`https://libreriacapullito.cl/scaneo_backend.php?ean=${props.codigo}`, {
            method: 'GET',
        })
        const data = await response.json();
        console.log("Response: " + JSON.stringify(data))
        if (data.length === 0 || data.error) {
            throw new Error('Producto no encontrado');
        }
        apiRespose.value = data[0];
    } catch (error) {
        errorMessage.value = {
            error: true,
            message: 'El producto que buscas no existe',
            hint: "Puedes revisar el código"
        };
        hasError.value = true;
    } finally {
        isLoading.value = false;
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
    height: 80%;
    width: 80%;
    background-color: #fff;
    padding: 15px;
    border-radius: 4px;
}

.close-action {
    width: 100%;
    height: 5%;
    justify-self: right;
    display: flex;
    justify-content: flex-end;
}

.product_data {
    height: 95%;
    position: relative;
}


.product_name {
    font-size: 1.25rem;
    font-weight: bold;
    margin-bottom: 10px;
}



.product_image {
    margin-bottom: 20px;
}

.price_container {
    display: flex;
    margin: 20px 0;
    justify-content: space-between;
    align-items: center;
    flex-direction: column;
}

.product_price_container {
    display: flex;
    column-gap: 10px;
}

.product_price {
    color: #086fcf;
    font-size: 35px;
    font-weight: 900;
}

.product_code {
    position: absolute;
    bottom: 0;
    font-size: 16px;
    color: #999;
    /* center with css */
    left: 50%;
    transform: translate(-50%, -50%);
}
</style>
  