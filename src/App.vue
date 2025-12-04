<template>
  <div id="app">
    <!-- Отображаем только наш генератор палитр -->
    <ColorPaletteGenerator />
  </div>
</template>

<script>
import ColorPaletteGenerator from './components/ColorPaletteGenerator.vue'

export default {
  name: 'App',
  components: {
    ColorPaletteGenerator
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  line-height: 1.6;
}
</style>