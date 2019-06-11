<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <ul>
      <to-do-item v-for="(item,index) in goodsList" :todo="item" :key="index"></to-do-item>
    </ul>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import ToDoItem from './components/ToDoItem.vue'

export default {
  name: 'app',
  components: {
    HelloWorld,
    ToDoItem
  },
  data(){
    return {
      goodsList:[
        {
          id:'1',
          text:'cola'
        },
        {
          id:'2',
          text:'candy'
        },
        {
          id:'3',
          text:'sugar'
        },
      ]
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
