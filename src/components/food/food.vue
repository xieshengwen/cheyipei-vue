<template>
  <transition name="move">
    <div class="detailWrapper" ref="detailWrapper" v-show="showDetail">
      <div class="foodDetail">
        <div class="image-header">
          <img :src="food.image">
          <div class="back" @click="showToggle">
            <i class="icon-arrow_lift"></i>
            </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="details">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">¥{{food.price}}</span><span class="old" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
          </div>
          <transition name="fade">
            <div class="buy" @click.stop.prevent="addCart($event)" v-show="!food.count">加入购物车</div>
          </transition>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food"></cartcontrol>
          </div>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="evaluation">
          <div class="title">
            商品评价
          </div>
          <div class="classify">
            <span v-for="(item, index) in classifyArr" class="item" :class="{'active':item.active,'bad':index==2,'badActive':item.active&&index==2}" @click="filterEvel(item)">
              {{item.name}}<span class="count">{{item.count}}</span>
            </span>
          </div>
          <div class="switch" @click="evelflag=!evelflag">
            <span class="icon-check_circle" :class="{'on':evelflag}"></span>
            <span class="text">只看有内容的评价</span>
          </div>
          <div class="evel-list">
            <ul>
              <li class="evel" v-for="evel in evelArr">
                <div class="userInfo">
                  <div class="time">{{evel.rateTime}}</div>
                  <div class="user">
                    <span>{{evel.username}}</span>
                    <span class="avatar"><img :src="evel.avatar" width="12" height="12"></span>
                  </div>
                </div>
                <div class="content">
                  <span class="icon" :class="evel.rateType?'icon-thumb_down':'icon-thumb_up'"></span>
                  <span class="text">{{evel.text}}</span>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import Vue from 'vue';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import split from 'components/split/split';
  const ALL = 2;

  export default {
    props: {
      food: {
        type: Object
      }
    },
    data() {
      return {
        showDetail: false,
        selectType: ALL,
        onlyContent: true,
        evelflag: true,
        classifyArr: [{
          name: '全部',
          count: this.food.ratings.length,
          active: true
        }, {
          name: '推荐',
          count: this.food.ratings.filter((data) => data.rateType === 0).length,
          active: false
        }, {
          name: '吐槽',
          count: this.food.ratings.filter((data) => data.rateType).length,
          active: false
        }]
      };
    },
    computed: {
      evelArr() {
        let selectIndex = 0;
        this.classifyArr.forEach((data, index) => {
          if (data.active) {
            selectIndex = index;
          }
        });
        if (this.detailWrapper) {
          this.$nextTick(() => {
            this.detailWrapper.refresh();
          });
        }
        return selectIndex ? this.food.ratings.filter((data) => this.evelflag ? data.rateType === selectIndex - 1 && data.text : data.rateType === selectIndex - 1) : this.food.ratings.filter((data) => this.evelflag ? data.text : true);
      }
    },
    methods: {
      show() {
        this.showDetail = true;
        this.selectType = ALL;
        this.onlyContent = true;
        this.$nextTick(() => {
          this._initScroll();
        });
      },
      showToggle() {
        this.showDetail = !this.showDetail;
      },
      _initScroll() {
        if (!this.detailWrapper) {
          this.detailWrapper = new BScroll(this.$refs.detailWrapper, {
            click: true
          });
        } else {
          this.detailWrapper.refresh();
        }
      },
      addCart(event) {
        if (!event._constructed) {
          return;
        }
        this.$set(this.food, 'count', 1);
        this.$root.eventHub.$emit('cart.add', event.target);
      },
      filterEvel(item) {
        this.classifyArr.forEach((data) => {
          data.active = false;
        });
        item.active = true;
      }
    },
    components: {
      cartcontrol,
      split
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin"

  .detailWrapper
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    z-index: 30
    width: 100%
    background: #fff
    transition all 0.4s ease
    &.move-enter-active,&.move-leave-active
      transform: translate3d(0,0,0);
    &.move-enter,&.move-leave-active
      transform: translate3d(100%,0,0)
    .image-header
      position: relative
      width: 100%;
      height: 0
      padding-top: 100%
      img
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
      .back
        position: absolute
        top: 10px
        left: 0
        .icon-arrow_lift
          display: block
          padding: 10px
          font-size: 20px
          color: #fff
  .foodDetail
    .content
      position relative
      box-sizing border-box
      width 100%
      padding 18px
      .title
        line-height: 14px
        margin-bottom: 8px
        font-size: 14px
        font-weight: 700
        color: rgb(7, 17, 27)
      .details
        margin-bottom: 18px
        height :10px
        line-height: 10px
        font-size:0
        .sell-count, .rating
          font-size: 10px
          color: rgb(147,153, 159)
        .sell-count
          margin-right: 12px
      .price
        font-weight: 700
        line-height: 24px
        .now
          margin-right: 8px
          font-size: 14px
          color: rgb(240, 20, 20)
        .old
          text-decoration: line-through
          font-size: 10px
          color: rgb(147, 153, 159)
      .cartcontrol-wrapper
        position: absolute
        right: 12px
        bottom: 12px
        z-index: 2
      .buy
        position: absolute
        right: 18px
        bottom: 18px
        z-index: 10
        height: 24px
        line-height: 24px
        padding: 0 12px
        box-sizing: border-box
        font-size: 10px
        border-radius: 12px
        color: #fff
        background: rgb(0,160,220)
        &.fade-enter-active, &.fade-leave-active
          transition: opacity .2s
        &.fade-enter, &.fade-leave-active
          opacity: 0
    .info
      padding: 18px
      .title
        line-height: 14px
        margin-bottom: 6px
        font-size: 14px
        color: rgb(7, 17, 27)
      .text
        line-height: 24px
        padding: 0 8px
        font-size: 12px
        color: rgb(77, 85, 93)
    .evaluation
      padding 18px 0
      position relative
      .title
        padding-left 18px
        font-size: 14px
        font-weight 500
        color: #07111b
      .classify
        padding 18px 0
        margin 0 18px
        border-bottom 1px solid rgba(7,17,27,0.1)
        .item
          display inline-block
          font-size 12px
          padding 8px 12px
          line-height 16px
          background rgba(0,160,220,0.2)
          color rgb(77,85,95)
          margin-right 8px
          .count
            font-size 8px
            padding-left 2px
          &.active
            color white
            background rgb(0,169,220)
          &.bad
            background rgba(77,85,93,0.2)
          &.badActive
            background #4d555d
      .switch
        font-size 12px
        width 100%
        padding 12px 0 12px 18px
        color rgb(147,153,159)
        border-bottom 1px solid rgba(7,17,27,0.1)
        .icon-check_circle
          font-size 24px
          vertical-align middle
          &.on
            color #00c850
      .evel-list
        margin 0 18px
        .evel
          padding 16px 0
          border-bottom 1px solid rgba(7,17,27,0.1)
          .userInfo
            display flex
            color rgb(147,153,159)
            font-size 10px
            line-height 12px
            .time
              flex 1
            .user
              flex 1
              text-align right
              .avatar
                img
                  padding-left 6px
                  border-radius 50%
          .content
            padding-top 6px
            .icon
              font-size 12px
              line-height 24px
              &.icon-thumb_up
                color rgb(0,160,220)
              &.icon-thumb_down
                color rgb(147,153,159)
            .text
              font-size 12px
              color rgb(7,17,27)
              line-height 16px
              padding-left 4px
</style>
