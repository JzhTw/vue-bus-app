<template>
  <div>
    <Nav />
    <div class="container">
      <div class="row">
        <div class="col-md-12">
          <div class="error-template">
            <h1>Oops!</h1>
            <h2>404 Not Found</h2>
            <div class="error-details">
              很抱歉, 發生了一點錯誤, 找不到此頁面!
            </div>
            <div class="error-actions">
              <router-link class="btn btn-primary btn-lg" to="/">
                回首頁
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer />
  </div>
</template>

<script>
import Nav from "@/components/Nav.vue";
import Footer from "@/components/Footer.vue";
export default {
  components: {
    Nav,
    Footer,
  },
};
</script>

<style scoped>
.error-template {
  padding: 40px 15px;
  text-align: center;
}
.error-actions {
  margin-top: 15px;
  margin-bottom: 15px;
}
.error-actions .btn {
  margin-right: 10px;
}
</style>