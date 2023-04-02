<template>
    <div class="loader-container">
        <svg class="loader" viewBox="0 0 50 50">
            <circle class="loader-circle" cx="25" cy="25" r="20"></circle>
        </svg>
    </div>
</template>

<script>
</script>


<style>
.loader-container {
    padding: 20px 10;
    display: flex;
    justify-content: center;
    align-items: center;
}

.loader {
    width: 50px;
    height: 50px;
    animation: rotate 2s linear infinite;
}

.loader-circle {
    fill: none;
    stroke: #1e90ff;
    stroke-width: 4;
    stroke-dasharray: 80px;
    stroke-dashoffset: 0px;
    animation: dash 1.5s ease-in-out infinite;
}

@keyframes rotate {
    100% {
        transform: rotate(360deg);
    }
}

@keyframes dash {
    0% {
        stroke-dashoffset: 80px;
    }

    50% {
        stroke-dashoffset: 20px;
    }

    100% {
        stroke-dashoffset: 80px;
    }
}
</style>