<template>
  <div class="ratings" ref="ratings">
    <div>
      <div class="overview">
        <div class="leftView">
          <h2 class="score">{{seller.score}}</h2>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}</div>
        </div>
        <div class="rightView">
          <div class="scoreWrapper">
            <span class="title">服务态度</span>
            <star :size="36" :score="seller.serviceScore"></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="scoreWrapper">
            <span class="title">商品评分</span>
            <star :size="36" :score="seller.foodScore"></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="scoreWrapper">
            <span class="title">送达时间</span>
            <span class="time">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <RatingSelect
        :selectType="selectType"
        :ratings="ratingList"
        @selectRateType="selectRateType"
        @toggleHasContent="toggleHasContent"></RatingSelect>
      <div class="ratingWrapper">
        <ul v-show="ratingList&&ratingList.length">
          <li v-for="(item,index) in ratingList" class="ratingItem border-1px"
              v-show="needShow(item.rateType,item.text)" :key="index">
            <div class="avatar">
              <img :src="item.avatar" width="28" height="28"/>
            </div>
            <div class="content">
              <h2 class="name">{{item.username}}</h2>
              <div class="starWrapper">
                <star :score="item.score"></star>
                <span class="delivery" v-show="item.deliveryTime">{{item.deliveryTime}}分钟送达</span>
              </div>
              <div class="text">{{item.text}}</div>
              <div class="ratingText">
                <i class="icon" :class="{'icon-thumb_up':item.rateType===0,'icon-thumb_down':item.rateType===1}"></i>
                <div class="recommend" v-show="item.recommend&&item.recommend.length">
                  <span v-for="(food,i) in item.recommend" class="food" :key="i">{{food}}</span>
                </div>
              </div>
              <div class="time">{{item.rateTime | formatDate}}</div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import Star from '../Star'
  import Split from '../Split'
  import RatingSelect from '../RatingSelect'
  import BScoll from 'better-scroll'
  import {getTemplateDate} from '../../common/js/date'

  export default {
    name: 'Ratings',
    props: ['seller'],
    data() {
      return {
        ratingList: [],
        ratingScoll: null,
        selectType: {
          type: 0,
          onlyContent: true,
          desc: {
            all: '全部',
            positive: '满意',
            negative: '不满意'
          }
        }
      }
    },
    created() {
      this.getRatingData()
    },
    methods: {
      getRatingData() {
        this.axios.get('/api/ratings')
          .then(res => {
            res = res.data
            if (res.errCode === 0) {
              this.ratingList = res.data
              this.$nextTick(() => {
                this._initScroll()
              })
            }
          })
          .catch(err => {
            console.log(err)
          })
      },
      _initScroll() {
        if (!this.ratingScoll) {
          this.ratingScoll = new BScoll(this.$refs.ratings, {
            click: true
          })
        } else {
          this.ratingScoll.refresh()
        }
      },
      needShow(type, text) {
        if (this.selectType.onlyContent && !text) {
          // 如果选择只显示有内容的评价,不管此条评论是好评还是差评只要没有内容直接return false
          return false
        }
        let result = true
        if (this.selectType.type !== 2) {
          result = type === this.selectType.type
        }
        return result
      },
      selectRateType(type) {
        this.selectType.type = type
        this._initScroll() // 刷新页面scroll
      },
      toggleHasContent() {
        this.selectType.onlyContent = !this.selectType.onlyContent
        this._initScroll() // 刷新页面scroll
      }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time)
        // 根据指定模板生成 对应模板的时间字符串
        return getTemplateDate(date, 'yyyy-MM-dd hh:mm')
      }
    },
    components: {
      Star,
      Split,
      RatingSelect
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../common/sass/mixin";

  .ratings {
    position: fixed;
    overflow: hidden;
    width: 100%;
    top: 175px;
    bottom: 46px;
    .overview {
      display: flex;
      padding: 18px 0;
      .leftView {
        flex: 0 0 137px;
        width: 137px;
        text-align: center;
        border-right: 1px solid rgba(7, 17, 27, 0.1);
        .score {
          margin-bottom: 6px;
          line-height: 28px;
          font-size: 24px;
          color: rgb(255, 153, 0);
        }
        .title {
          margin-bottom: 8px;
          line-height: 12px;
          font-size: 12px;
          color: rgb(7, 17, 27);
        }
        .rank {
          margin-bottom: 6px;
          line-height: 10px;
          font-size: 10px;
          color: rgb(147, 153, 159);
        }
      }
      .rightView {
        flex: 1;
        padding: 0 24px;
        font-size: 0;
        .scoreWrapper {
          margin-bottom: 8px;
          &:last-child {
            margin-bottom: 0;
          }
          .title {
            display: inline-block;
            vertical-align: top;
            font-size: 12px;
            line-height: 17px;
            color: rgb(7, 17, 27)
          }
          .star {
            display: inline-block;
            margin: 0 12px;
            vertical-align: top;
          }
          .score {
            line-height: 17px;
            font-size: 12px;
            color: rgb(255, 153, 0);
          }
          .time {
            margin-left: 12px;
            line-height: 17px;
            font-size: 12px;
            color: rgb(147, 153, 159);
          }
        }
      }
    }
    .ratingWrapper {
      padding: 0 18px;
      font-size: 0;
      .ratingItem {
        position: relative;
        display: flex;
        padding: 18px 0;
        width: 100%;
        @include border-1px(rgba(7, 17, 27, 0.1));
        .avatar {
          flex: 0 0 28px;
          margin-right: 12px;
          width: 28px;
          img {
            border-radius: 50%;
          }
        }
        .content {
          flex: 1;
          .name {
            margin-bottom: 4px;
            line-height: 12px;
            font-size: 10px;
            color: rgb(7, 17, 27);
          }
          .starWrapper {
            margin-bottom: 6px;
            .star {
              display: inline-block;
              margin-right: 6px;
            }
            .delivery {
              line-height: 12px;
              font-weight: 200;
              font-size: 10px;
              color: rgb(147, 153, 159);
            }
          }
          .text {
            margin-bottom: 8px;
            line-height: 18px;
            font-size: 12px;
            color: rgb(7, 17, 27);
          }
          .ratingText {
            display: flex;
            line-height: 16px;
            .icon {
              flex: 0 0 20px;
              font-size: 12px;
              &.icon-thumb_up {
                color: rgb(0, 160, 220);
              }
              &.icon-thumb_down {
                color: rgb(183, 187, 191);
              }
            }
            .recommend {
              flex: 1;
              font-size: 9px;
              color: rgb(147, 153, 159);
              .food {
                display: inline-block;
                margin: 0 0 4px 8px;
                padding: 0 6px;
                border-radius: 2px;
                border: 1px solid rgba(7, 17, 27, 0.1);
              }
            }
          }
          .time {
            position: absolute;
            right: 0;
            top: 18px;
            line-height: 24px;
            font-size: 10px;
            font-weight: 200;
            color: rgb(147, 153, 159);
          }
        }
      }
    }
  }
</style>
