<template>
  <div class="home">
    <Header />
    <Field />
    <Footer />
  </div>
</template>

<script>
import Header from "./components/Header.vue";
import Footer from "./components/Footer.vue";
import Field from "./components/Field.vue";
export default {
  components: {
    Header,
    Footer,
    Field,
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
