<template>
  <div class="picker-calendar">
    <div class="select-calendar">
      <div class="year-month-day">
        <div class=" today iconfont icon-jintian" @click.prevent="clickToday"></div>
        <div class=" cur-date" @click.prevent="pickerCalendar" ><span>{{year}}年{{month}}月</span></div>
        <div class=" iconfont icon-rili"  @click.prevent="pickerCalendar"></div>
      </div>
      <div class="date-scroll-container">
        <div class="slipe-wrapper">
          <swiper :options="swiperOption" ref="mySwiper">
            <swiper-slide class="week-row" v-for="(week, index) in calendarData" :key="index">
              <div
                class="date-item"
                v-for="(item, index) in week">
                <div class="date-week">{{item.weekCn}}</div>
                <div class="date-day"
                  :class="[item.isToday? 'today': '' ,
                      ( item.isSelect && !item.isToday)? 'select': '',
                      item.isCurMonth? '': 'not-cur-month',
                      ]"
                    @click.prevent.stop="changeDay(item.stamp)">
                  <p class="solar">
                    <span class="is-leap" v-if="item.lDate.isLeap && item.lDate.IDayCn==='初一'">闰</span>
                    <span :class="[item.week === 6 || item.week === 0 ?'weekend': '']">{{item.date}}</span>
                  </p>
                  <p class="lunar">
                    <span class="festival" v-if="item.lDate.isFestival">{{item.lDate.festival[0]}}</span>
                    <span class="term" v-else-if="item.lDate.isTerm">{{item.lDate.Term}}</span>
                    <span v-else-if="item.lDate.IDayCn==='初一'">{{item.lDate.IMonthCn.replace('闰','')}}</span>
                    <span v-else>{{item.lDate.IDayCn}}</span>
                  </p>
                </div>
              </div>
            </swiper-slide>
          </swiper>
        </div>
      </div>
    </div>
    <popup v-model="showPopup">
      <div class="dialog-calendar">
        <div @click="showPopup=false" class="close">
          取消
        </div>
        <div @click="sure" class="sure" >
          确定
        </div>
        <div style="display: block;clear:both;"></div>
      </div>
      <datetime-view v-model="selectTime"></datetime-view>
    </popup>
  </div>
</template>

<script>
import 'swiper/dist/css/swiper.css'
import moment from 'moment';
import {Popup,DatetimeView} from 'vux'
import JDatePickerScript from './jDatePicker.js'
import { swiper, swiperSlide } from 'vue-awesome-swiper'
const weekCn = ['日', '一', '二', '三', '四', '五', '六']
const currentMonth = moment();
  export default{
    data() {
      return {
        year: 0, // 年份
        month: 0, // 月份
        curIndex: 0, // 滑动到当前的索引
        TodayIndex: 0, // 今天所在的索引
        calendarData: [], // 日历数据
        lastWeekLength: 0, // 上个月份的星期长度
        curWeekLength: 0, // 当前月的星期长度
        nextWeekLength: 0, // 下个月的星期长度
        currentMonth, // 当前日期
        selectDay: [], // 选中的日期
        swiperOption: {
          on:{
              slideChangeTransitionEnd: ()=> {
              this.curIndex = this.$refs.mySwiper.swiper.activeIndex;
          }}
        },
        showPopup: false, // 显示弹窗
        selectTime: '', // 弹窗选中时间
      }
    },
    computed: {
      swiper() {
        return this.$refs.mySwiper.swiper;
      }
    },
    // 监听年月改变
    watch: {
        curIndex: function(){
          // 当前索引值在上个月
          if(this.curIndex < this.lastWeekLength) {
            this.month = this.currentMonth.month() == 0 ? this.month = 12 : this.currentMonth.month();
            this.year =  this.currentMonth.month() == 0 ? moment(this.currentMonth).year() - 1 : moment(this.currentMonth).year();
          }else if( this.curIndex > (this.lastWeekLength + this.curWeekLength)) { // 当前索引值在下个月
            this.month = this.currentMonth.month() == 11 ? this.month = 1 : this.currentMonth.month() + 2;
            this.year =  this.currentMonth.month() == 11 ? moment(this.currentMonth).year() + 1 : moment(this.currentMonth).year();
          }else {
            this.month = this.currentMonth.month() + 1;
            this.year =  moment(this.currentMonth).year();
          }
        }
    },
    mounted() {
      this.calendarData = this.handleCalendar(moment());
      this.month = moment().month() + 1;
      this.year = moment().year();
      this.TodayIndex = (moment().date())/7 + this.lastWeekLength;
      this.swiper.slideTo(Math.round(this.TodayIndex), 100, false);
    },
    methods: {
      // 点击今天
      clickToday() {
        this.currentMonth = moment();
        this.selectDay = []; // 其他选中的非当天的其他的日期清空
        this.calendarData = this.handleCalendar(moment());
        this.month = moment().month() + 1;
        this.year = moment().year();
        this.swiper.slideTo(Math.round(this.TodayIndex), 100, false);
      },
      // 选择日期
      pickerCalendar(){
        this.showPopup = true;
      },
      // 改变日期
      changeDay(stamp) {
        // 添加选中的非当天的日期
        if(this.selectDay.length > 0){
          this.selectDay.splice(0,1);
          this.selectDay.push(stamp);
        }else{
          this.selectDay.push(stamp);
        }
        const currentYear = new Date(stamp).getFullYear()
        const currentMonth = new Date(stamp).getMonth() + 1
        if (this.year !== currentYear) this.year = currentYear
        if (this.month !== currentMonth) this.month = currentMonth
        this.currentMonth = moment(new Date(stamp))
        this.calendarData = this.handleCalendar(this.currentMonth);
      },
      // 确定日期
      sure() {
        this.showPopup = false;
        let time = this.selectTime.split('-');
        this.year = time[0];
        this.month = parseInt(time[1]) < 10 ? parseInt(time[1]): time[1] ;
        this.currentMonth = moment(this.selectTime);
         // 添加选中的非当天的日期
        if(this.selectDay.length > 0){
          this.selectDay.splice(0,1);
          this.selectDay.push(new Date(this.selectTime).getTime());
        }else{
          this.selectDay.push(new Date(this.selectTime).getTime());
        }
        this.calendarData = this.handleCalendar(this.currentMonth);
        // 日历滚动当前选中的日期
        let index = Math.ceil(parseInt(time[2])/7) + Math.round(this.lastWeekLength);
        this.swiper.slideTo(index - 1, 100, false);
      },
      handleCalendar(currentMonth) {
        let curMonth = currentMonth.clone();
        let lastMonth = moment(moment(curMonth).subtract(1, 'months')).startOf('month') // 上个月的第一天
        let nextMonth = moment(moment(curMonth).add(1, 'months')).startOf('month') // 下个月的第一天
        this.lastWeekLength = (moment(lastMonth).daysInMonth() + lastMonth.day()) /7;
        this.curWeekLength = moment(curMonth).daysInMonth()/7;
        this.nextWeekLength = (moment(nextMonth).daysInMonth() + nextMonth.day())/7;
        const dateRange = lastMonth.daysInMonth() + curMonth.daysInMonth() + nextMonth.daysInMonth(); // 日历滚动范围，此为显示三个月
        let calendars = [];
        let days = [];
        lastMonth.subtract(lastMonth.day(), 'd');
        for(let i = 0; i < dateRange + lastMonth.day() + nextMonth.day(); i++) {
          const obj = {
              stamp:new Date(lastMonth).getTime(),
              isToday: lastMonth.isSame(new Date(), 'day'),
              isSelect: this.selectDay.length>0 && lastMonth.isSame(new Date(this.selectDay[0]), 'day'),
              isCurMonth: lastMonth.isSame(currentMonth, 'month'),
              lDate: JDatePickerScript.calendar.getLunar(lastMonth.toISOString()),
              week: new Date(lastMonth).getDay(),
              weekCn: weekCn[new Date(lastMonth).getDay()],
              date: new Date(lastMonth).getDate(),
              day: lastMonth.date()
            }
            lastMonth.add(1, 'd') // 加一天
            days.push(obj)
        }
        // 日期分割成周
        for(let i = 0; i < dateRange/7; i++) {
          calendars.push(days.splice(0, 7))
        }
        console.log(calendars)
        return calendars;
      }
    },
    components: {
      swiper,
      swiperSlide,
      Popup,DatetimeView
    }
  }
</script>

<style lang="scss" scoped>
 @import '../../assets/iconfont/iconfont.css';
  .picker-calendar{
    // height: 100%;
    .year-month-day {
      display: flex;
      height: 50px;
      line-height: 50px;
      padding: 0 14px;
      .today {
        float: left;
        font-size: 2.3rem;
        color: #ee4e4e;
      }
      .cur-date {
        width: 100%;
        text-align: center;
        font-weight: bold;
        font-size: 1.5rem;
      }
      .icon-rili {
        float: right;
        font-size: 2rem;
      }
    }
    .date-scroll-container{
      overflow: hidden;
      width: 100%;
      height: 75px;
      margin-top: 2px;
      padding-bottom: 4px;
      -webkit-overflow-scrolling: touch;
      background-color: #f5f5f5;
      .slipe-wrapper {
        height: 75px;
      }
      .week-row {
        width: 100%;
        float:left;
        display: flex;
      }
      .date-item{
        flex: 1;
        text-align: center;
        float: left;
        display: inline-block;
        // font-size: 18rem;
        color: #999;
        // height: 6.7rem;

        .date-week{
          font-size: 1.1rem;
          height: 30px;
          line-height: 30px;
        }
        .date-day{
          margin: 0 auto;
          height: 45px;
          width: 4rem;
          // line-height: 4rem;
          border-radius: 50%;
          .solar {
            width: 100%;
            height: 22px;
            line-height: 22px;
            font-weight: bold;
            font-size: 1.8rem;
            color: #333;
            .is-leap {
              color: #ee4e4e;
            }
            .weekend {
              color: #999;
            }
          }
          .lunar {
            width: 100%;
            height: 15px;
            line-height: 15px;
            font-size: 1rem;
            .festival, .term {
              color: #ee4e4e;
            }
          }
          &.today {
            width: 40px;
            height: 40px;
            border: 1px solid #ee4e4e;
            .solar {
              color: #ee4e4e;
            }
            .lunar {
              color: #ee4e4e;
            }
          }
          &.select {
            width: 42px;
            height: 42px;
            background: #0E82F7;
            box-shadow: 0 2px 4px 0 rgba(14,130,247,0.26);
            color: #fff;
            .solar {
              color: #fff;
              .weekend {
                color: #fff;
              }
            }
            .lunar {
              .festival, .term {
                color: #fff;
              }
            }
          }
          &.clicked{
            background-color: #283858;
            color: #fff;
            &.today{
              background-color: #1478F0;
              color: #fff;
            }
          }
          &.today{
            color: #1478F0;
          }
        }
      }
    }
    .dialog-calendar {
      height: 30px;
      line-height: 30px;
      font-size: 1.5rem;
      .close{
        float: left;
        padding-left: 10px;
        color: #828282;
      }
      .sure{
        float: right;
        padding-right: 10px;
        color:#FF9900;
      }
    }

  }
</style>
