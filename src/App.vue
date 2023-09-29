<template>
  <MainPage />
</template>

<script>
import MainPage from "./components/MainPage.vue";

export default {
  name: "App",
  components: {
    MainPage,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

.centered {
    display: flex;
    align-items: center;
    justify-content: center;
}

.left-flex {
    display: flex;
    align-items: center;
    justify-content: flex-start;
}

.right-flex {
    display: flex;
    align-items: center;
    justify-content: flex-end;
}

.bg-danger {
  background-color: #D65104!important;
}

.bg-success {
  background-color: #02A6A6!important;
}

.btn-success {
  background-color: #02A6A6!important;
}


</style>
