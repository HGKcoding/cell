<template>
  <div id="app">
    <m-header :seller="seller"></m-header>
    <tab></tab>
    <keep-alive>
      <router-view :seller="seller" :goods="goods" :ratings="ratings"></router-view>
    </keep-alive>
  </div>
</template>

<script>
import MHeader from 'components/m-header/m-header'
import Tab from 'components/tab/tab'
// import { urlParse } from 'common/js/util'
// const ERR_OK = 0

export default {
  data () {
    return {
      seller: {
        flag: false
        /* id: (() => {
          let queryParam = urlParse()
          return queryParam.id
        })() */
      },
      goods: [],
      ratings: []
    }
  },
  created () {
    let data = require('../static/data.json')
    this.ratings = data.ratings
    this.goods = data.goods
    this.seller = Object.assign({}, this.seller, data.seller)
    this.seller.flag = true
    /* this.$http.get('/api/seller?id=' + this.seller.id).then((res) => {
      res = res.data
      if (res.errno === ERR_OK) {
        this.seller = Object.assign({}, this.seller, res.data)
      }
    }) */
  },
  name: 'App',
  components: {
    MHeader,
    Tab
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">

</style>
