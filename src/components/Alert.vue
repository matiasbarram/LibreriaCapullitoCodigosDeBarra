<template>
    <div class="alert alert-light" role="alert">
        <span style="font-weight: 600;">{{ message }}</span>
        {{ subMessage }}
    </div>
</template>

<script setup>
const props = defineProps({
    message: {
        type: String,
        required: true
    },
    subMessage: {
        type: String,
        required: false
    },
    style: {
        type: Object,
        required: false
    }
})
</script>