<template>
  <div id="app">
    <con-main></con-main>
  </div>
</template>

<script>

import main from './views/Main'
import axios from 'axios'
export default {
  data(){
    return {
    }
  },
  components: {
    'con-main': main
  },
  methods: {
    getData(){
      
    }
  },
  created(){

  }
}
</script>

<style>

</style>
