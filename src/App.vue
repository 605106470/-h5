<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tabs border-1px">
      <div class="tab-item">
        <router-link to="/">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <router-view :seller="seller"></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
import Header from './components/header/header.vue';
const ERR_OK = 0;
export default {
  name: 'App',
  components: {
    'v-header': Header
  },
  data() {
    return {
      seller: {}
    };
  },
  created() {
    this.$http.get('/api/seller').then(res => {
      if (res.body.errno === ERR_OK) {
        this.seller = res.body.data;
      }
    });
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "./common/stylus/mixin.styl"

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  color: #2c3e50;
  // margin-top: 60px;
  .tabs{
    display:flex;
    width:100%;
    height:40px;
    line-height :40px;
    border-1px(rgba(7,17,27,0.1))
    .tab-item{
      flex:1
      text-align:center;
      a{
        display:block;
        font-size:14px;
        color:rgb(77,85,93)
      }
      .router-link-exact-active{
        color:red;
      }
    }
  }
}
</style>
