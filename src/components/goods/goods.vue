<template>
  <div class="goods">
    <scroll :probe-type="probeType" :listen-scroll="listenScroll">
    <div class="menu-wrapper">
      <ul>
        <li v-for="(item, index) in goods" :key="index" class="menu-item" :class="{'current': currentIndex === index}" @click="selectMenu(index)">
          <span class="text border-1px">
            <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{ item.name }}
          </span>
        </li>
      </ul>
    </div>
    </scroll>
    <scroll ref="listview" @scroll="scroll" :probe-type="probeType" :listen-scroll="listenScroll">
    <div class="foods-wrapper">
      <ul>
        <li v-for="(item, index) in goods" class="food-list" ref="listGroup" :key="index">
          <h1 class="title">{{ item.name }}</h1>
          <ul>
            <li @click="selectFood(food)" v-for="(food, index) in item.foods" class="food-item border-1px" :key="index">
              <div class="icon">
                <img width="57" height="57" :src="food.icon">
              </div>
              <div class="content">
                <h2 class="name">{{ food.name }}</h2>
                <p class="desc"> {{ food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{ food.sellCount}}份</span><span>好评率{{ food.rating }}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{ food.price}}</span><span v-show="food.oldPrice" class="old">￥{{ food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="addFood" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    </scroll>
    <shopcart @drop="addFood" ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    <food @drop="addFood" :food="selectedFood" ref="food"></food>
  </div>
</template>

<script>
import Scroll from 'base/scroll/scroll'
import Shopcart from 'components/shopcart/shopcart'
import Cartcontrol from 'components/cartcontrol/cartcontrol'
import Food from 'components/food/food'
// const ERR_OK = 0
export default {
  props: {
    seller: {},
    goods: {
      default () {
        return []
      }
    }
  },
  data () {
    return {
      currentIndex: 0,
      // goods: [],
      scrollY: 0,
      classMap: ['decrease', 'discount', 'special', 'invoice', 'guarantee'],
      selectedFood: {}
    }
  },
  created () {
    this.listHeight = []
    this.probeType = 3
    this.listenScroll = true
    this.$nextTick(() => {
      this._calculateHeight()
    })
    /* this.$http.get('/api/goods').then((res) => {
      res = res.data
      if (res.errno === ERR_OK) {
        this.goods = res.data
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      }
    }) */
  },
  methods: {
    scroll (pos) {
      this.scrollY = pos.y
    },
    _calculateHeight () {
      let foodList = this.$refs.listGroup
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodList.length; i++) {
        height += foodList[i].clientHeight
        this.listHeight.push(height)
      }
    },
    selectMenu (index) {
      if (index < 0) { // 处理边界
        index = 0
      } else if (index > this.listHeight.length - 2) {
        index = this.listHeight.length - 2
      }
      this.scrollY = -this.listHeight[index]
      this.$refs.listview.scrollToElement(this.$refs.listGroup[index]) // scrollToElement()
    },
    addFood (target) {
      this._drop(target)
    },
    _drop (target) {
      this.$nextTick(() => { /* 写一个回调，防止移动端同时开两个动画会卡 */
        this.$refs.shopcart.drop(target)
      })
    },
    selectFood (food) {
      this.selectedFood = food
      this.$refs.food.show()
    }
  },
  computed: {
    selectFoods () {
      let foods = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  components: {
    Scroll,
    Shopcart,
    Cartcontrol,
    Food
  },
  watch: {
    scrollY (newY) {
      const listHeight = this.listHeight
      // 当滚动到顶部，newY > 0
      if (newY > 0) {
        this.currentIndex = 0
        return
      }
      // 在中间滚动
      for (let i = 0; i < listHeight.length - 1; i++) {
        let height1 = listHeight[i]
        let height2 = listHeight[i + 1]
        if (-newY >= height1 && -newY < height2) { // 滑到底 || 在区间之内
          this.currentIndex = i
          this.diff = height2 + newY
          return
        }
      }
      // 当滚动到底部，且 -newY大于最后一个元素的上限
      this.currentIndex = this.listHeight.length - 2
    }
  }
}
</script>

<style lang="stylus" scoped>
@import "~common/stylus/mixin"
.goods
  display flex
  position absolute
  top 174px
  bottom 46px
  width 100%
  overflow hidden
  .menu-wrapper
    flex 0 0 80px
    width 80px
    background #f3f5f7
    .menu-item
      display table
      height 54px
      width 56px
      padding 0 12px
      line-height 14px
      &.current
        position relative
        z-index 10
        margin-top -1px
        background #fff
        font-weight 700
        .text
          border-none()
      .icon
        display inline-block
        vertical-align top
        width 12px
        height 12px
        margin-right 2px
        background-size 12px
        background-repeat no-repeat
        &.decrease
          bg-image('decrease_3')
        &.discount
          bg-image('discount_3')
        &.guarantee
          bg-image('guarantee_3')
        &.invoice
          bg-image('invoice_3')
        &.special
          bg-image('special_3')
      .text
        display table-cell
        width 56px
        vertical-align middle
        border-1px(rgba(7, 17, 27, 0.1))
        font-size 12px
  .foods-wrapper
    flex 1
    .title
      padding-left 14px
      height 26px
      line-height 26px
      border-left 2px solid #d9dde1
      font-size 12px
      color rgb(147, 153, 159)
      background #f3f5f7
    .food-item
      display flex
      margin 18px /* 外边距塌陷 */
      padding-bottom 18px
      border-1px(rgba(7, 17, 27, 0.1))
      &:last-child
        border-none()
        margin-bottom 0
      .icon
        flex 0 0 57px
        margin-right 10px
      .content
        flex 1
        .name
          margin 2px 0 8px 0
          height 14px
          line-height 14px
          font-size 14px
          color rgb(7, 17, 27)
        .desc, .extra
          line-height 10px
          font-size 10px
          color rgb(147, 153, 159)
        .desc
          line-height 12px
          margin-bottom 8px
        .extra
          .count
            margin-right 12px
        .price
          font-weight 700
          line-height 24px
          .now
            margin-right 8px
            font-size 14px
            color rgb(240, 20, 20)
          .old
            text-decoration line-through /* 中间横杠 */
            font-size 10px
            color rgb(147, 153, 159)
        .cartcontrol-wrapper
          position absolute
          right 0
          bottom 12px
</style>
