<template>
  <div id="app">
    <InvoiceQR />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import InvoiceQR from './components/InvoiceQR.vue'
import 'purecss'

@Component({
  components: {
    InvoiceQR
  }
})
export default class App extends Vue {}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  margin: 0 auto;
  width: 400px;
  max-width: 100%;
}
</style>
