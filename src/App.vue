<template>
  <TheCalendar />
</template>

<script>
import TheCalendar from "./components/TheCalendar.vue";

export default {
  name: "App",
  components: {
    TheCalendar,
  },
  data() {
    return {
    }
  },
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
