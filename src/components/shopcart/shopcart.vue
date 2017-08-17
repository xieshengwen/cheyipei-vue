<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="listToggle">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">¥{{totalPrice}}</div>
          <div class="desc">另需配送费¥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right" @click.stop.prevent="pay">
          <div class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop" v-for="(ball,index) in balls" :key="index">
          <div class="ball" v-show="ball.show">
            <div class="inner inner-hook"></div>
          </div>
        </transition>
      </div>
      <transition name="transHeight">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="setEmpty()">清空</span>
          </div>
          <div class="list-content" ref="foodlist">
            <ul>
              <li class="food" v-for="food in selectFoods">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price * food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade-backdrop">
      <div class="backdrop" v-show="showBackdrop" @click="hideBackdrop"></div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import backdrop from 'components/backdrop/backdrop';
  import BScroll from 'better-scroll';

  export default {
    props: {
      selectFoods: {
        type: Array,
        default() {
          return [
             {
              price: 20,
              count: 1
            }
          ];
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        balls: [{
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }],
        dropBalls: [],
        listShow: false
      };
    },
    created() {
      this.$root.eventHub.$on('cart.add', this.drop);
    },
    computed: {
      showBackdrop() {
        if (this.listShow && this.totalPrice) {
          return true;
        }
        this.listShow = false;
        return false;
      },
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return `¥${this.minPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差¥${diff}元起送`;
        } else {
          return `去结算`;
        }
      },
      payClass() {
       if (this.totalPrice < this.minPrice) {
          return 'not-enough';
       } else {
         return 'enough';
       }
      }
    },
    methods: {
      drop(el) {
        for (let i = 0, l = this.balls.length; i < l; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            this.dropBalls.push(ball);
            return;
          }
        }
      },
      setEmpty() {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
      },
      pay() {
        if (this.totalPrice < this.minPrice) {
          return;
        }
        window.alert(`支付${this.totalPrice}`);
      },
      hideBackdrop() {
        this.listShow = false;
      },
      _initScroll() {
        this.foodlistScroll = new BScroll(this.$refs.foodlist, {
          click: true
        });
      },
      listToggle() {
        if (!this.selectFoods.length) {
          return;
        }
        this.listShow = !this.listShow;
        if (this.listShow) {
          this.$nextTick(() => {
            if (!this.foodlistScroll) {
              this._initScroll();
            } else {
              this.foodlistScroll.refresh();
            }
          });
        }
      },
      beforeDrop(el) {
        let count = this.balls.length;
        while (count--) {
          let ball = this.balls[count];
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect();
            let x = rect.left - 32;
            let y = -(window.innerHeight - rect.top - 22);
            el.style.display = '';
            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
            el.style.transform = `translate3d(0,${y}px,0)`;
            let inner = el.querySelector('.inner-hook');
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
            inner.style.transform = `translate3d(${x}px,0,0)`;
          }
        }
      },
      dropping(el, done) {
        el.offsetHeight; // 触发浏览器重绘，offsetWidth、offsetTop等方法都可以触发
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transform = 'translate3d(0,0,0)';
          let inner = el.querySelector('.inner-hook');
          inner.style.webkitTransform = 'translate3d(0,0,0)';
          inner.style.transform = 'translate3d(0,0,0)';
          el.addEventListener('transitionend', done);
        });
      },
      afterDrop(el) {
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
          el.style.display = 'none';
        }
      }
    },
    components: {
      cartcontrol,
      backdrop
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin"

  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display: flex
      height: 48px
      background: #141d27
      color: rgba(255, 255, 255, 0.4)
      font-size: 0
      .content-left
        flex: 1
        .logo-wrapper
          display: inline-block
          vertical-align: top
          position: relative
          top: -10px
          margin: 0 12px
          padding: 6px
          width: 56px
          height 56px
          box-sizing: border-box
          border-radius: 50%
          background: #141d27
          .logo
            width: 100%
            height: 100%
            border-radius: 50%
            background: #2b343c
            &.highlight
              background: rgb(0, 160, 220)
            text-align: center
            .icon-shopping_cart
              font-size: 24px
              line-height 44px
              color: #80858a
              &.highlight
                color: #fff
          .num
            position: absolute
            top: 0
            right: 0
            width: 24px
            height: 16px
            line-height: 16px
            text-align: center
            border-radius: 16px
            font-size: 9px
            font-weight: 700
            color: #fff
            background: rgb(240, 20, 20)
            box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          line-height: 24px
          padding-right: 12px
          box-sizing: border-box
          border-right: 1px solid rgba(255, 255, 255, 0.1)
          font-size: 16px
          font-weight: 700
          &.highlight
            color: #fff
        .desc
          display: inline-block
          vertical-align: top
          line-height: 24px
          margin: 12px 0 0 12px
          line-height: 24px
          font-size: 10px
      .content-right
        flex: 0 0 105px
        width: 105px
        .pay
          height 48px
          line-height: 48px
          text-align: center
          font-size: 12px
          font-weight: 700
          background: #2b333b
          &.not-enough
            background:#2b333b
          &.enough
            background: #00b43c
            color: #fff
    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        &.drop-enter,&.drop-enter-active
          transition: all 0.4s cubic-bezier(0.49,-0.29,0.75,0.41)
          .inner
            width: 16px
            height: 16px
            border-radius: 50%
            background: rgb(0,160,220)
            transition: all 0.4s linear
    .shopcart-list
      position: absolute
      top: 0
      left: 0
      width: 100%
      background: white
      transform: translate3d(0,-100%,0)
      z-index: -1
      &.transHeight-enter-active,&.transHeight-leave-active
        transition: all 0.5s
      &.transHeight-enter,&.transHeight-leave-active
        transform: translate3d(0,0,0)
      .list-header
        height: 40px
        line-height: 40px
        background: #f3f5f7
        border-bottom: 1px solid rgba(7,17,27,0.1)
        .title
          display: inline-block
          font-size: 14px
          font-weight: 200
          color: rgb(7,17,27)
          padding-left: 18px
        .empty
          position: absolute
          right: 8px
          font-size: 12px
          color: rgb(0,160,220)
          padding: 0 10px
      .list-content
        padding: 0 18px
        max-height: 217px
        overflow: hidden
        .food
          position: relative
          padding: 12px 0
          box-sizing: border-box
          border-bottom-1px(rgba(7, 17, 27, 0.1))
          .name
            font-size: 14px
            line-height: 24px
            color: rgb(7,17,27)
          .price
            position: absolute
            right: 90px
            bottom: 12px
            line-height: 24px
            font-size: 14px
            font-weight: 700
            color: rgb(240,20,20)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 3px
  .backdrop
    position fixed
    top 0
    bottom 0
    left 0
    right 0
    background rgba(7,17,27,0.6)
    backdrop-filter blur(10px)
    z-index 40
    &.fade-backdrop-enter-active,&.fade-backdrop-leave-active
      transition opacity 0.5s
    &.fade-backdrop-enter,&.fade-backdrop-leave-active
      opacity 0
</style>
