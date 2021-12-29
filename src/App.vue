<template>
  <div id="app" class="container-fluid p-0">
    <div>
      <router-view />
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
};
</script>


<style lang="scss">
body {
  background: url("~@/assets/main.jpg");
  background-repeat: repeat;
  height: 100%;
  padding: 0;
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  color: #2c3e50;
}
#nav {
  background: #61a68a;
}

.bus__txt {
  color: #61a68a;
}
</style>
