<template>
  <div class="addBooking-wrap">
    <div class="tab">
      <div class="tab-item" :class="{'active': recordType == 0}" @click="tabSwitch(0)">支出</div>
      <div class="tab-item" :class="{'active': recordType == 1}" @click="tabSwitch(1)">收入</div>
    </div>
    <swiper
      :indicator-dots="indicatorDots"
      :autoplay="autoplay"
      class="swiper"
      @change="swiperChange"
      :current="recordType"
    >
      <swiper-item class="swiper-item swiper-item-1" :class="{'isIphoneX': isIphoneX}">
        <scroll-view style="height: 100%" scroll-y="true">
          <div class="pay-income-note">
            <input v-model="payIncomeNote" placeholder="为你的支出写点什么吧QvQ" />
          </div>
          <ul class="list-wrap icon-wrap">
            <li
              class="list-item icon-item with-title"
              v-for="(item, index) in pList"
              :key="item.id"
              :class="{active:paySelectedIndex == index}"
            >
              <div class="circle" :data-index="index" @click="showKeyboard(item, index)">
                <i
                  class="iconfont"
                  :class="item.iconClassName"
                  style="line-height: 110rpx; text-align: center; font-size: 50rpx"
                ></i>
              </div>
              <p class="icon-name">{{item.categoryName}}</p>
            </li>
            <li class="list-item icon-item">
              <div class="circle" @click="goToEdit">
                <i
                  class="iconfont icon-jiahao"
                  style="line-height: 110rpx; text-align: center; font-size: 50rpx"
                ></i>
              </div>
            </li>
          </ul>
        </scroll-view>
      </swiper-item>

      <!-- 第二页 -->
      <swiper-item class="swiper-item swiper-item-2" :class="{'isIphoneX': isIphoneX}">
        <scroll-view style="height: 100%" scroll-y="true">
          <div class="pay-income-note">
            <input v-model="payIncomeNote" placeholder="为你的收入写点什么吧QvQ" />
          </div>
          <ul class="list-wrap icon-wrap">
            <li
              class="list-item icon-item with-title"
              v-for="(item, index) in iList"
              :key="item.id"
              :class="{active:incomeSelectedIndex == index}"
            >
              <div class="circle" :data-index="index" @click="showKeyboard(item, index)">
                <i
                  class="iconfont"
                  :class="item.iconClassName"
                  style="line-height: 110rpx; text-align: center; font-size: 50rpx"
                ></i>
              </div>
              <p class="icon-name">{{item.categoryName}}</p>
            </li>
            <li class="list-item icon-item">
              <div class="circle" @click="goToEdit">
                <i
                  class="iconfont icon-jiahao"
                  style="line-height: 110rpx; text-align: center; font-size: 50rpx"
                ></i>
              </div>
            </li>
          </ul>
        </scroll-view>
      </swiper-item>
    </swiper>

    <keyboard @bindSubmit="bindSubmit" :reset.sync="resetKeyBoard"></keyboard>
  </div>
</template>

<script>
import tabBar from '@/components/tabBar'
import timePicker from '@/components/timePicker'
import keyboard from '@/components/keyboard'

// api
import { addRecord, getCategoryList } from '@/utils/api'

import utils from '@/utils'

export default {
  data(){
    return {
      indicatorDots: false,
      autoplay: false,
      inputBottom: 0, //input初始高度
      // showInput: false,
      selectDate: '',
      price: null,
      recordType: 0, // 默认支出
      payCategoryList: [],
      incomeCategoryList: [],
      currentIconClassName: '',
      categoryId: '',
      categoryName: '',
      chooseCalendar: false,

       // 支出相关
      paySelectedIndex: null,

      // 收入相关
      incomeSelectedIndex: null,
      resetKeyBoard:false,

      // 收入支出备注
      payIncomeNote: '',
    }
  },
  components: {
    tabBar,
    timePicker,
    keyboard
  },
  methods: {
    bindSubmit(val){
      console.log('接受键盘',val)
      this.selectDate = val.selectDate
      if (!parseFloat(val.price)) {
        wx.showToast({
          title: '请输入金额',
          icon: 'none',
          duration: 2000
        })
        return
      }
      if (this.currentIconClassName == '') {
        wx.showToast({
          title: '请选择分类',
          icon: 'none',
          duration: 2000
        })
        return
      }
      let params = {
        userId: wx.getStorageSync("user_info").user.id,
        categoryId: this.categoryId,
        billYear: this.recordYear,
        billMonth: parseInt(this.recordMonth),
        billDay: parseInt(this.recordDay),
        billType: this.recordType,
        price: val.price,
        categoryName: this.categoryName,
        icon: this.currentIconClassName,
        payIncomeNote: this.payIncomeNote,
      }
      addRecord(params).then(res => {
        if (res.code == 1) {
          this.resetBooking()
          
          this.$store.commit('ACCLIST_NEED_REFRESH')
          wx.switchTab({
            url: '/pages/index/main'
          })
        }
      })
    },

    bindCalendar(){
      this.chooseCalendar = true
    },
    resetBooking(){
      this.currentIconClassName = ''
      this.categoryId = '';
      this.categoryName = ''
      this.paySelectedIndex = null;
      this.incomeSelectedIndex = null;
      this.price = null;
      this.chooseCalendar = false;
      this.resetKeyBoard = true;
      this.payIncomeNote = '';
    },
    showKeyboard(item,index){
      this.currentIconClassName = item.iconClassName;
      this.categoryId = item.id;
      this.categoryName = item.categoryName;

      if (this.recordType == 0) {
        this.paySelectedIndex = index
      } else if (this.recordType == 1) {
        this.incomeSelectedIndex = index
      }
      
      // this.showInput = true
    },
    goToEdit(){
      wx.navigateTo({
        url: '/pages/categoryList/main?type=' + this.recordType
      })
    },
    swiperChange(e){
      this.recordType = e.mp.detail.current
      this.resetBooking()
    },
    tabSwitch(index){
      this.recordType = index
    },
    submit(){
    },
    async getCategoryList(recordType){
      try {
        let res = await getCategoryList({recordType})
        if (recordType == 0) {
          this.$store.commit('GET_PAYLIST', res.data.categoryList)
        } else if (recordType == 1) {
          this.$store.commit('GET_INCOMELIST', res.data.categoryList)
        }
      } catch (error) {
        console.log(error)
      }

    }
  },
  watch: {
    recordType(){
       wx.setNavigationBarTitle({
        title: `${this.recordType == 0? '支出':'收入'}记账`,
      })
    }
  },
  created () {
  },
  mounted(){
    this.$store.dispatch('getCategoryList',0)
    this.$store.dispatch('getCategoryList',1)
  },
  computed: {
    isIphoneX(){
        return this.$store.getters.isIphoneX
    },
    recordYear(){
      return utils.getTodayDate(this.selectDate).year
    },
     recordMonth(){
      return utils.getTodayDate(this.selectDate).month
    },
     recordDay(){
      return utils.getTodayDate(this.selectDate).day
    },
    iList(){
      return this.$store.getters.iList
    },
    pList(){
      return this.$store.getters.pList
    },
  },
  onUnload(){
    this.recordType = 0
  },
}
</script>

<style lang="stylus" scoped>
inputLineHeight = 110px;

.tab {
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100px;
  background: themeColor;
  z-index: 100;

  .tab-item {
    flex: 1;
    height: 100px;
    box-sizing: border-box;
    color: fontColor;
    font-size: 30px;
    text-align: center;
    line-height: 100px;
  }
}

.swiper {
  height: 100vh;

  .swiper-item {
    box-sizing: border-box;
    padding-top: 100px;
    padding-bottom: keyBoardHeight;

    &.isIphoneX {
      padding-bottom: keyBoardHeight + isPhoneXBottom;
    }

    // .list-wrap
    // display flex
    // flex-wrap wrap
    // align-content flex-start
    // justify-content flex-start
    // .list-item
    // width (750px / 4)
    // height (750px / 4)
    // display flex
    // align-items center
    // justify-content center
    // .circle
    // width 110px
    // height 110px
    // background #eee
    // border-radius 50%
    &.swiper-item-1, &.swiper-item-2 {
      background: #fff;
    }
  }

  .pay-income-note {
    height: 60px;
    margin-top: 10px;
    padding: 15px 0px 5px 30px;
    color: blue;
    border-radius: 20px;
    border-color: black;
    border: 2px dashed;
  }
}

.input-wrap {
  display: flex;
  height: inputLineHeight;
  position: fixed;
  bottom: 200px;
  // border-top 1px solid #b5b5b5
  box-shadow: 0px -5px 10px #b5b5b5;
  width: 100%;
  font-size: 36px;
  color: fontColor;

  .digit-input {
    flex: 1;
    height: 100%;
    padding-left: 50px;
    padding-right: 50px;
  }

  .select-date {
    box-sizing: border-box;
    width: 100px;
    height: 100%;
    border-right: 1px solid #b5b5b5;
    border-left: 1px solid #b5b5b5;
    position: relative;
    line-height: inputLineHeight;
    text-align: center;
    font-size: 20px;
    padding: 0 10px;

    .format-select-date {
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      justify-content: center;

      span {
        display: inline-block;
        height: 40px;
        line-height: 40px;
        font-size: 26px;

        &.year {
          font-size: 20px;
        }
      }
    }
  }

  .submit {
    width: 200px;
    height: 100%;
    background: themeColor;
    text-align: center;
    line-height: 110px;
  }
}
</style>
