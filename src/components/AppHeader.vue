<template>
  <header class="header-mobile">
    <div style="display: flex">
      <button class="hamburger-menu">
        <svg viewBox="0 0 24 24" width="24" height="24" class="hamburger-icon">
          <path fill="currentColor" d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z" />
        </svg>
      </button>
      <section class="header-title">
        Escanea tu producto
        <span class="header-subtitle">para más información</span>
      </section>
    </div>
    <div class="logo">
      <!-- <img class="logo" src="https://libreriacapullito.cl/img/logoCapullito.png" alt="Libreria capullito"> -->
    </div>
  </header>
</template>
  
<style scoped>
.header-title {
  font-weight: 600;
  font-size: 16px;
  line-height: normal;
  margin: 0 14px;
}

.header-subtitle {
  font-size: 14px;
  display: block;
  font-weight: 400;
}

.header-mobile {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.logo {
  width: 65px;
}

.hamburger-menu {
  width: 30px;
  height: 30px;
  background-color: transparent;
  border: none;
  outline: none;
  cursor: pointer;
}

.hamburger-icon {
  color: #17a2b8
}
</style>

<script setup>
import { ref } from 'vue'

</script>