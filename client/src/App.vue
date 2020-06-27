<template>
  <div id="app">
    <h1 class="heading-text">todos</h1>
    <b-container>
      <b-row class="justify-content-md-center">
        <todo class="todo" />
      </b-row>
    </b-container>
  </div>
</template>

<script>
import todo from "./components/todo";

export default {
  name: "App",
  components: {
    todo
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.heading-text {
  font-size: 60px;
  font-weight: 600;
}
.todo {
  width: 50%;
}
</style>
