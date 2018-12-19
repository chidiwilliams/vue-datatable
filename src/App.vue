<template>
  <div id="app">
    <DataTable/>
  </div>
</template>

<script>
import DataTable from './components/DataTable.vue'

export default {
  name: 'app',
  components: {
    DataTable
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Karla');

#app {
  font-family: 'Karla', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
