<template>
  <header class="header-mobile menu-container">
    <div style="display: flex">
      <button class="hamburger-menu menu-toggle" aria-expanded="false" aria-label="Toggle menu" @click="toggleMenu">
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

  <nav class="menu" :class="{ open: isMenuOpen }" ref="target">
    <div class="main-menu">
      <button class="hamburger-menu menu-toggle" aria-expanded="false" aria-label="Toggle menu" @click="toggleMenu">
        <svg viewBox="0 0 24 24" width="24" height="24" class="hamburger-icon">
          <path fill="currentColor" d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z" />
        </svg>
      </button>
      <ul>
        <li><a class="menu-ancle" href="https://libreriacapullito.cl/">Nuestros productos</a></li>
      </ul>
    </div>
    <div class="footer-menu">
      <img class="logo" src="https://libreriacapullito.cl/img/logoCapullito.png" alt="Libreria capullito" />
    </div>

  </nav>
</template>

<script setup>
import { onClickOutside } from '@vueuse/core';
import { ref } from 'vue'

const isMenuOpen = ref(false);
const target = ref(null)
onClickOutside(target, (event) => {
  if (isMenuOpen.value == true) {
    toggleMenu()
  }
})


function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value;
  console.log("is open: " + isMenuOpen.value)
}

</script>

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



.menu-container {
  display: block;
}

.menu-toggle {
  position: relative;
  z-index: 2;
  display: block;
  width: 40px;
  height: 40px;
  background: transparent;
  border: none;
  cursor: pointer;
  transition: transform 0.3s ease-in-out;
}

.menu-toggle:hover .menu-icon::before,
.menu-toggle:hover .menu-icon::after {
  background-color: #555;
}

.menu-icon,
.menu-icon::before,
.menu-icon::after {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 25px;
  height: 2px;
  background-color: #999;
  transition: background-color 0.3s ease-in-out;
}

.menu-icon::before,
.menu-icon::after {
  content: '';
  display: block;
}

.menu-icon::before {
  transform: translateY(-8px);
}

.menu-icon::after {
  transform: translateY(8px);
}

.menu-toggle[aria-expanded="true"] .menu-icon::before {
  transform: translateY(0) rotate(45deg);
}

.menu-toggle[aria-expanded="true"] .menu-icon::after {
  transform: translateY(0) rotate(-45deg);
}

.menu-toggle[aria-expanded="true"] .menu-icon {
  background-color: transparent;
}

.menu {
  position: absolute;
  display: grid;
  grid-template-rows: 1fr auto;
  top: 0;
  left: -240px;
  z-index: 2;
  width: 240px;
  height: 100%;
  padding: 20px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  transition: transform 0.3s ease-in-out;
}

.menu ul {
  margin-top: 20px !important;
  list-style: none;
  margin: 0;
  padding: 0;
}

.menu li {
  margin-bottom: 10px;
}

.menu a {
  display: block;
  padding: 10px;
  color: #333;
  text-decoration: none;
  transition: background-color 0.3s ease-in-out;
}

.menu a:hover {
  background-color: #eee;
}

.menu.open {
  transform: translateX(200px);
}

.menu-ancle {
  font-size: 16px;
  font-weight: 600;
}
</style>
