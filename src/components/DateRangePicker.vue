<template>
    <div style="position: relative; display: inline-block;">
        <div class="form-control" @click="togglePicker">
            <slot
                    name="input"
                    :startDate="start"
                    :endDate="end"
                    :ranges="ranges"
            >
                <i class="glyphicon glyphicon-calendar fa fa-calendar"></i>&nbsp;
                <span>{{startText}} - {{endText}}</span>
                <b class="caret"></b>
            </slot>
        </div>
        <transition name="slide-fade" mode="out-in">
            <div
                    class="daterangepicker dropdown-menu ltr show-ranges d-block"
                    :class="pickerStyles()"
                    v-if="open"
                    v-on-clickaway="clickAway"
            >
                <div class="headline d-flex justify-content-between" v-if="headline">
                  <h5 class="title">{{headlineTitle}}</h5>
                  <div class="icons-wrapper">
                    <i class="ocpx-icon-boxycross"></i>
                    <i class="ocpx-icon-minus"></i>
                  </div>
                </div>
                <div class="calendars d-flex">
                    <calendar-ranges
                            @clickRange="clickRange"
                            :ranges="ranges"
                    ></calendar-ranges>
                    <div class="calendars-container d-flex flex-wrap">
                      <div class="calendars-wrapper d-flex">
                        <div class="drp-calendar left">
                          <div class="daterangepicker_input hidden-xs" v-if="false">
                            <input class="input-mini form-control" type="text" name="daterangepicker_start"
                            :value="startText"/>
                            <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
                          </div>
                          <div class="calendar-table">
                            <calendar
                            :monthDate="monthDate"
                            :locale="locale"
                            :start="start" :end="end"
                            :minDate="min" :maxDate="max"
                            :showDropdowns="true"
                            @nextMonth="nextMonth" @prevMonth="prevMonth"
                            @dateClick="dateClick" @hoverDate="hoverDate"
                            ></calendar>
                          </div>
                          <div class="slider-wrapper d-flex align-items-center" v-if="hours">
                            <span class="d-block">Hour</span>
                            <slider
                            ref="HoursSliderLeft"
                            v-model="leftHour"
                            @drag="leftHoursSliderDrag"
                            :max="23"
                            :min="0"
                            />
                          </div>
                          <div class="slider-wrapper d-flex align-items-center" v-if="hours">
                            <span class="d-block">Minute</span>
                            <slider
                            ref="MinutesSliderLeft"
                            v-model="leftMinute"
                            @drag="leftMinutesSliderDrag"
                            :max="59"
                            :min="0"
                            />
                          </div>
                        </div>
                        <div class="drp-calendar right hidden-xs">
                          <div class="daterangepicker_input" v-if="false">
                            <input class="input-mini form-control" type="text" name="daterangepicker_end"
                            :value="endText"/>
                            <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
                          </div>
                          <div class="calendar-table">
                            <calendar
                            :monthDate="nextMonthDate"
                            :locale="locale"
                            :start="start" :end="end"
                            :minDate="min" :maxDate="max"
                            :showDropdowns="true"
                            @nextMonth="nextMonth" @prevMonth="prevMonth"
                            @dateClick="dateClick" @hoverDate="hoverDate"
                            ></calendar>
                          </div>
                          <div class="slider-wrapper d-flex align-items-center" v-if="hours">
                            <span class="d-block">Hour</span>
                            <slider
                            ref="HoursSliderRight"
                            v-model="rightHour"
                            @drag="rightHoursSliderDrag"
                            :max="23"
                            :min="0"
                            />
                          </div>
                          <div class="slider-wrapper d-flex align-items-center" v-if="hours">
                            <span class="d-block">Minute</span>
                            <slider
                            ref="MinutesSliderRight"
                            v-model="rightMinute"
                            @drag="rightMinutesSliderDrag"
                            :max="59"
                            :min="0"
                            />
                          </div>
                        </div>
                      </div>
                      <div class="footer-wrapper w-100 d-flex justify-content-between">
                        <div class="inputs-wrapper d-flex justify-content-center align-items-center" v-if="footerInputs">
                          <div class="input-wrapper d-flex">
                            <span class="icon-wrapper d-flex align-items-center justify-content-center">
                              <i :class="footerInputIconClassName"></i>
                            </span>
                            <input type="text" class="input" v-model="startInputText" readonly>
                          </div>
                          <span class="separator">
                            <i class="ocpx-icon-minus"></i>
                          </span>
                          <div class="input-wrapper d-flex">
                            <span class="icon-wrapper d-flex align-items-center justify-content-center">
                              <i :class="footerInputIconClassName"></i>
                            </span>
                            <input type="text" class="input" v-model="endInputText" readonly>
                          </div>
                        </div>
                        <div class="drp-buttons d-flex">
                          <button
                          class="applyBtn btn btn-sm btn-success"
                          :disabled="in_selection"
                          type="button"
                          @click="clickedApply"
                          >
                          <i class="ocpx ocpx-icon-save"></i>
                          {{locale.applyLabel}}</button>
                          <!-- <button
                          class="cancelBtn btn btn-sm btn-default"
                          type="button"
                          @click="open=false"
                          >{{locale.cancelLabel}}</button> -->
                        </div>
                      </div>
                    </div>
                </div>


            </div>
        </transition>
    </div>
</template>

<script>
  import moment from 'moment'
  import Calendar from './Calendar.vue'
  import CalendarRanges from './CalendarRanges'
  import {nextMonth, prevMonth} from './util'
  import {mixin as clickaway} from 'vue-clickaway'
  import Slider from 'vue-slider-component'

  export default {
    components: {Calendar, CalendarRanges, Slider},
    mixins: [clickaway],
    props: {
      minDate: [String, Object],
      maxDate: [String, Object],
      footerInputs: {
        type: Boolean,
        default(){
          return false;
        }
      },
      headline: {
        type: Boolean,
        default(){
          return false;
        }
      },
      headlineTitle: {
        type: String,
        default(){
          return '';
        }
      },
      footerInputIconClassName: {
        type: String,
        default(){
          return '';
        }
      },
      hours: {
        type: Boolean,
        default(){
          return false;
        }
      },
      localeData: {
        type: Object,
        default () {
          return {}
        },
      },
      startDate: {
        default () {
          return moment()
        }
      },
      endDate: {
        default () {
          return moment()
        }
      },
      ranges: {
        type: Object,
        default () {
          return {
            'Today': [moment(), moment()],
            'Yesterday': [moment().subtract(1, 'days'), moment().subtract(1, 'days')],
            'This month': [moment().startOf('month'), moment().endOf('month')],
            'This year': [moment().startOf('year'), moment().endOf('year')],
            'Last week': [moment().subtract(1, 'week').startOf('week'), moment().subtract(1, 'week').endOf('week')],
            'Last month': [moment().subtract(1, 'month').startOf('month'), moment().subtract(1, 'month').endOf('month')],
          }
        }
      },
      opens: {
        type: String,
        default: 'center'
      }
    },
    data () {
      let default_locale = {
        direction: 'ltr',
        format: 'DD-MM-YYYY hh:mm',//moment.localeData().longDateFormat('L'),
        separator: ' - ',
        applyLabel: 'Set range',
        cancelLabel: 'Cancel',
        weekLabel: 'W',
        daysOfWeek: moment.weekdaysMin(),
        monthNames: moment.months(),
        years: [],
        firstDay: moment.localeData().firstDayOfWeek()
      }

      // let data = { locale: _locale }
      let data = {locale: {...default_locale, ...this.localeData}}

      data.monthDate = new Date(this.startDate)
      data.start = new Date(this.startDate)
      data.end = new Date(this.endDate)
      data.in_selection = false
      data.open = false
      data.leftHour = 0
      data.leftMinute = 0
      data.rightHour = 0
      data.rightMinute = 0
      // update day names order to firstDay
      if (data.locale.firstDay !== 0) {
        let iterator = data.locale.firstDay
        while (iterator > 0) {
          data.locale.daysOfWeek.push(data.locale.daysOfWeek.shift())
          iterator--
        }
      }
      return data
    },
    methods: {
      nextMonth () {
        this.monthDate = nextMonth(this.monthDate)
      },
      prevMonth () {
        this.monthDate = prevMonth(this.monthDate)
      },
      dateClick (value) {
        if (this.in_selection) {
          this.in_selection = false
          this.end = new Date(value)
          if (this.end < this.start) {
            this.in_selection = true
            this.start = new Date(value)
          }
        } else {
          this.in_selection = true
          this.start = new Date(value)
          this.end = new Date(value)
        }
      },
      hoverDate (value) {
        let dt = new Date(value)
        if (this.in_selection && dt > this.start)
          this.end = dt
      },
      togglePicker () {
        this.open = !this.open
      },
      pickerStyles () {
        return {
          'show-calendar': this.open,
          opensright: this.opens === 'right',
          opensleft: this.opens === 'left',
          openscenter: this.opens === 'center'
        }
      },
      clickedApply () {
        this.open = false
        this.$emit('update', {startDate: this.start, endDate: this.end})
      },
      clickAway () {
        if (this.open) {
          this.open = false
        }
      },
      clickRange (value) {
        this.start = new Date(value[0])
        this.end = new Date(value[1])
        this.monthDate = new Date(value[0])
        this.clickedApply()
      },
      leftHoursSliderDrag(value){
        let {
          updateHours,
          leftHour,
          start,
          end
        } = this

        if (leftHour < 24 && leftHour > 0) {
          start.setHours(leftHour)
          this.$emit('update', {
            startDate: start,
            endDate: end
          })
        }
      },
      leftMinutesSliderDrag(){
        let {
          updateMinutes,
          leftMinute,
          start,
          end
        } = this

        if (leftMinute < 60 && leftMinute > 0) {
          start.setMinutes(leftMinute)
          this.$emit('update', {
            startDate: start,
            endDate: end
          })
        }
      },
      rightHoursSliderDrag(){
        let {
          updateHours,
          rightHour,
          start,
          end
        } = this

        if (rightHour < 24 && rightHour > 0) {
          end.setHours(rightHour)
          this.$emit('update', {
            startDate: start,
            endDate: end
          })
        }
      },
      rightMinutesSliderDrag(){
        let {
          updateMinutes,
          rightMinute,
          start,
          end
        } = this

        if (rightMinute < 60 && rightMinute > 0) {
          end.setMinutes(rightMinute)
          this.$emit('update', {
            startDate: start,
            endDate: end
          })
        }
      },
    },
    computed: {
      nextMonthDate () {
        return nextMonth(this.monthDate)
      },
      startText () {
        return moment(this.start).format(this.locale.format)
      },
      startInputText() {
        return moment(this.start).format('DD-MM-YYYY hh:mm');
      },
      endInputText () {
        return moment(this.end).format('DD-MM-YYYY hh:mm')
      },
      endText () {
        return moment(this.end).format(this.locale.format)
      },
      min () {
        return this.minDate ? new Date(this.minDate) : null
      },
      max () {
        return this.maxDate ? new Date(this.maxDate) : null
      }
    },
    watch: {
      startDate (value) {
        this.start = new Date(value)
      },
      endDate (value) {
        this.end = new Date(value)
      }
    }
  }

</script>

<style lang="scss">
    @import '../assets/daterangepicker.css';
</style>

<style lang="scss" scoped>
    .reportrange-text {
        background: #fff;
        cursor: pointer;
        padding: 5px 10px;
        border: 1px solid #ccc;
        width: 100%;
    }

    .daterangepicker{
        width: auto;
    }

    div.daterangepicker.opensleft {
        top: 35px;
        right: 10px;
        left: auto;
    }

    div.daterangepicker.openscenter {
        top: 35px;
        right: auto;
        left: 50%;
        transform: translate(-50%, 0);
    }

    div.daterangepicker.opensright {
        top: 35px;
        left: 10px;
        right: auto;
    }

    /* Enter and leave animations can use different */
    /* durations and timing functions.              */
    .slide-fade-enter-active {
        transition: all .2s ease;
    }

    .slide-fade-leave-active {
        transition: all .1s cubic-bezier(1.0, 0.5, 0.8, 1.0);
    }

    .slide-fade-enter, .slide-fade-leave-to
        /* .slide-fade-leave-active for <2.1.8 */
    {
        transform: translateX(10px);
        opacity: 0;
    }
    /deep/ .vue-slider-component.vue-slider-horizontal{
      flex: 1;
    }
</style>
