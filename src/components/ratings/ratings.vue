<template>
  <div class="ratings" ref="ratings">
    <div class="ratings-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <star :size="36" :score="seller.serviceScore"></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <star :size="36" :score="seller.foodScore"></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper">
            <span class="title">送达时间</span>
            <span class="delivery">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <div class="evaluation">
        <div class="classify">
            <span v-for="(item,index) in classifyArr" class="item" :class="{'active':item.active,'bad':index==2,'badActive':item.active&&index==2}" @click="filterEvel(item)">
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
              <div class="avatar">
                <img :src="evel.avatar" width="28" height="28">
              </div>
              <div class="content">
                <div class="user">
                  <span class="name">{{evel.username}}</span>
                  <span class="rateTime">{{evel.rateTime | time}}</span>
                </div>
                <div class="star-wrapper">
                  <star :size="24" :score="evel.score"></star>
                  <span class="deliveryTime">{{evel.deliveryTime}}分钟送达</span>
                </div>
                <div class="text">
                  {{evel.text}}
                </div>
                <div class="recommend">
                  <span class="icon icon-thumb_up" v-show="evel.recommend.length"></span>
                  <span class="dish" v-for="dish in evel.recommend">{{dish}}</span>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import '../../filter/time.js';
  import star from 'components/star/star';
  import split from 'components/split/split';
  import BScroll from 'better-scroll';
  import axios from 'axios';

  export default {
     props: {
      seller: {
        type: Object
      }
    },
    components: {
      star,
      split
    },
    data() {
      return {
        ratings: [],
        classifyArr: [{
          name: '全部',
          count: 0,
          active: true
        }, {
          name: '推荐',
          count: 0,
          active: false
        }, {
          name: '吐槽',
          count: 0,
          active: false
        }],
        evelflag: true
      };
    },
    created() {
      this._init();
    },
    computed: {
      evelArr() {
        let selectIndex = 0;
        this.classifyArr.forEach((data, index) => {
          if (data.active) {
            selectIndex = index;
          }
        });
        if (this.scroll) {
          this.$nextTick(() => {
            this.scroll.refresh();
          });
        }
        return selectIndex ? this.ratings.filter((data) => this.evelflag ? data.rateType === selectIndex - 1 && data.text : data.rateType === selectIndex - 1) : this.ratings.filter((data) => this.evelflag ? data.text : true);
      }
    },
    methods: {
      _init() {
        axios.get('static/data.json').then((response) => {
          this.ratings = response.data.ratings;
          this._initClassifyArr();
          this.$nextTick(() => {
            this.scroll = new BScroll(this.$refs.ratings, {
              click: true
            });
          });
        });
      },
      _initClassifyArr() {
        this.classifyArr.forEach((data, index) => {
          if (index) {
            data.count = this.ratings.filter((temp) => temp.rateType === index - 1).length;
          } else {
            data.count = this.ratings.length;
          }
        });
      },
      filterEvel(item) {
        this.classifyArr.forEach((data) => {
          data.active = false;
        });
        item.active = true;
      }
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .ratings
    position: absolute
    top: 174px
    bottom: 0
    left: 0
    width: 100%
    overflow: hidden
    .overview
      display: flex
      padding: 18px 0
      .overview-left
        flex: 0 0 137px
        padding: 6px 0
        width: 137px
        border-right: 1px solid rgba(7, 17, 27, 0.1)
        text-align: center
        @media only screen and (max-width :320px)
          flex: 0 0 120px
          width: 120px
        .score
          margin-bottom: 6px
          line-height: 28px
          font-size: 24px
          color: rgb(255, 153, 0)
        .title
          margin-bottom: 8px
          line-height: 12px
          font-size: 12px
          color: rgb(7, 17, 27)
        .rank
          line-height: 10px
          font-size: 10px
          color: rgb(147, 153, 159)
      .overview-right
        flex: 1
        padding: 6px 0 6px 24px
        @media only screen and (max-width :320px)
          padding-left: 6px
        .score-wrapper
          margin-bottom: 8px
          font-size: 0
          .title
            display: inline-block
            line-height: 18px
            vertical-align: top
            font-size: 12px
            color: rgb(7, 17 ,27)
          .star
            display: inline-block
            margin: 0 12px
            vertical-align: top
          .score
            display: inline-block
            line-height: 18px
            vertical-align: top
            font-size: 12px
            color: rgb(255, 133, 0)
        .delivery-wrapper
          font-size: 0
          .title
            line-height: 18px
            font-size: 12px
            color: rgb(7, 17 ,27)
          .delivery
            margin-left: 12px
            font-size: 12px
            color: rgb(147, 153, 159)
  .evaluation
    padding 18px 0
    position relative
    .classify
      padding-bottom 18px
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
    .evel
      display flex
      padding 18px 0
      margin 0 18px
      border-bottom 1px solid rgba(7,17,27,0.1)
      .avatar
        flex 0 0 28px
        margin-right 12px
        img
          border-radius 50%
      .content
        flex 1
        .user
          font-size 10px
          color rgb(7,17,27)
          line-height 12px
          .rateTime
            position absolute
            font-weight 200
            right 18px
            color rgb(147,153,159)
        .star-wrapper
          font-size 0
          padding-top 4px
          margin-bottom 6px
          .star
            display inline-block
          .deliveryTime
            font-size 10px
            padding-left 6px
            font-weight 200
            color rgb(147,153,159)
        .text
          font-size 12px
          color rgb(7,17,27)
          line-height 18px
        .recommend
          padding-top 4px
          .icon
            font-size 12px
            color rgb(0,160,220)
            line-height 16px
          .dish
            display inline-block
            font-size 9px
            color rgb(147,153,159)
            line-height 16px
            border 1px solid rgba(7,17,27,0.1)
            padding 2px 6px
            margin-right 8px
            white-space normal
            margin-top 4px
</style>
