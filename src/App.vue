<template>
  <div id="app">
    <Header/>
    <DropForm/>
  </div>
</template>

<script>
import DropForm from './components/DropForm.vue'
import Header from './components/Header.vue'

export default {
  name: 'App',
  components: {
    DropForm,
    Header
  }
}
</script>

<style>

</style>
