<template>
  <div id="app" class="wrapper">
    <h1>{{ title }}</h1>
    <todo></todo>
  </div>
</template>

<script>
import ToDo from './components/ToDo.vue'

export default {
  name: 'app',
  components: {
    'todo': ToDo,
  },
  data () {
    return {
      title: 'The Friendly Todo List'
    }
  }
}
</script>

<style lang="scss">
html, body, .wrapper {
  font-family: freight-text-pro, serif;
  color: #FFDEA6 !important;
  background-color: #828DAB;
  min-height: 100vh;
  margin: 0;
}

input {
  font-family: 'Source Code Pro', monospace !important;
  background-color: #828DAB;
  border: none;
  border-bottom: 1px solid #FFDEA6;
  color: #FFDEA6;
  font-size: 1.3rem !important;
}

* {
  box-sizing: border-box;
}
.wrapper {
  max-width: 500px;
  margin: 0 auto;
  padding-top: 6rem;
}

#app {
  font-size: 1.3rem;
}

h1 {
  font-family: freight-text-pro, serif;
  font-size: 3rem !important;
  font-weight: 700 !important;
  margin-bottom: 1rem !important;
}

</style>
