<template>
  <div id="app">
    <Header></Header>
    <router-view></router-view>
    <Footer v-show="$route.meta.isFooterShow"></Footer>
  </div>
</template>

<script>
import Header from "./components/Header/Header.vue";
import Footer from "./components/Footer/Footer.vue";

export default {
  name: "",
  components: { Header, Footer },
  created() {
    this.$store.dispatch("categoryList");
  },
};
</script>

<style lang="less">
</style>
