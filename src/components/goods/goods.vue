<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li
          v-for="(item, index) in goods"
          :key="index"
          class="menu-item"
          :class="{ current: currentIndex === index }"
          @click="selectMenu(index, $event)"
        >
          <span class="text border-1px">
            <span
              v-show="item.type > 0"
              class="icon"
              :class="classMap[item.type]"
            ></span
            >{{ item.name }}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li
          v-for="(item, index) in goods"
          :key="index"
          class="food-list food-list-hook"
        >
          <h1 class="title">{{ item.name }}</h1>
          <ul>
            <li
              v-for="(food, index1) in item.foods"
              :key="index1"
              class="food-item"
            >
              <div class="icon">
                <img width="57" height="57" :src="food.icon" alt="" />
              </div>
              <div class="content">
                <h2 class="name">{{ food.name }}</h2>
                <p class="desc">{{ food.description }}</p>
                <div class="extra">
                  <span class="count">月售{{ food.sellCount }}份</span
                  ><span>好评率{{ food.rating }}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{ food.price }}</span>
                  <span class="old" v-show="food.oldPrice"
                    >￥{{ food.oldPrice }}</span
                  >
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @cart-add="cartAdd" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart
      ref="shopCart"
      :delivery-price="seller.deliveryPrice"
      :min-price="seller.minPrice"
      :select-foods="selectFoods"
    ></shopcart>
  </div>
</template>

<script>
import BScroll from 'better-scroll';
import shopcart from '@/components/shopcart/shopcart';
import cartcontrol from '@/components/cartcontrol/cartcontrol';
const ERR_OK = 0;
export default {
  components: {
    shopcart: shopcart,
    cartcontrol: cartcontrol
  },
  data() {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0
    };
  },
  props: {
    seller: {
      type: Object
    }
  },
  created() {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    this.$http.get('/api/goods').then(res => {
      if (res.body.errno === ERR_OK) {
        this.goods = res.body.data;
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      }
    });
  },
  computed: {
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i;
        }
      }
      return 0;
    },
    selectFoods() {
      let foods = [];
      this.goods.forEach(good => {
        good.foods.forEach(food => {
          if (food.count) {
            foods.push(food);
          }
        });
      });

      return foods;
    }
  },
  methods: {
    _initScroll() {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      });
      this.foodsScroll.on('scroll', pos => {
        this.scrollY = Math.abs(Math.round(pos.y));
      });
    },
    _calculateHeight() {
      let foodList = this.$refs.foodsWrapper.getElementsByClassName(
        'food-list-hook'
      );
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    },
    selectMenu(index, event) {
      // 因为这里的BScroll插件会屏蔽原本dom的各种事件，所以把参数设置为click：true后，意味着我们自己派发了一个click时间，但浏览器模式下本来也有一个click事件，在浏览器模式下事件又会执行两次，为了屏蔽浏览器事件执行两次，采用event._constructed(浏览器模式下的时间没有改属性，会为false)
      if (!event._constructed) {
        return;
      }
      let foodList = this.$refs.foodsWrapper.getElementsByClassName(
        'food-list-hook'
      );
      let el = foodList[index];
      this.foodsScroll.scrollToElement(el, 500);
    },
    cartAdd(target) {
      this._drop(target);
    },
    _drop(target) {
      this.$refs.shopCart.drop(target);
    }
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus" scope>
@import "../../common/stylus/mixin";
.goods
  display:flex
  position:absolute
  top:174px
  bottom:46px
  width:100%
  overflow:hidden
  .menu-wrapper
    flex:0 0 80px
    width:80px
    background:#f3f5f7
    .menu-item
      display :table
      height:54px
      width:56px
      line-height:14px
      padding:0 12px
      &.current
        position:relative
        margin-top:-1px
        z-index:10
        background:#fff
        font-weight:700
        .text
          border-none()
      .icon
        display:inline-block
        vertical-align :top
        width:12px
        height:12px
        margin-right :2px
        background-size:12px 12px
        background-repeat:no-repeat
        &.decrease
          bg-image('../../resource/img/decrease_3')
        &.discount
          bg-image('../../resource/img/discount_3')
        &.guarantee
          bg-image('../../resource/img/guarantee_3')
        &.invoice
          bg-image('../../resource/img/invoice_3')
        &.special
          bg-image('../../resource/img/special_3')
      .text
        display:table-cell
        width:56px
        vertical-align:middle
        border-1px(rgba(7,17,27,0.1))
        font-size:12px
  .foods-wrapper
    flex:1
    .title
      padding-left:14px
      height:26px
      line-height:26px
      border-left:2px solid #d9dde1
      font-size:12px
      color:rgb(147,153,159)
      background:#f3f5f7
    .food-item
      display :flex
      margin:18px
      padding-bottom:18px
      border-1px(rgba(7,17,27,0.1))
      &:last-child
        border-none()
        margin-bottom:0
      .icon
        flex:0 0 57px
        margin-right:10px
      .content
        flex:1
        .name
          margin:2px 0 8px 0
          height:14px
          line-height :14px
          font-size :14px
          color:rgb(7,17,27)
        .desc,.extra
          line-height :10px
          font-size:10px
          color:rgb(147,153,159)
        .desc
          margin-bottom:8px
          line-height :12px
        .extra
          .count
            margin-right :12px
        .price
          font-weight :700
          line-height:24px
          .now
            margin-right :18px
            font-size:14px
            color:rgb(240,20,20)
          .old
            text-decoration :line-through
            font-size :10px
            color:rgb(147,153,159)
        .cartcontrol-wrapper
            position:absolute
            right:0
            bottom:12px
</style>
