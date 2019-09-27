<template>
  <div
    :class="[calendarClass, 'vdp-datepicker__calendar']"
    v-show="showMonthView"
    :style="calendarStyle"
    @mousedown.prevent
  >
    <slot name="beforeCalendarHeader"></slot>
    <header class="vdp-datepicker__header vdp-datepicker__header_full-date" v-if="showDateHeader">
      <span class="vdp-datepicker__control" @click="showDayCalendar">{{ currDay }}</span>
      <span class="vdp-datepicker__control vdp-datepicker__control_active">{{ currMonthName }}</span>
      <span class="vdp-datepicker__control" @click="showYearCalendar">{{ currYearName }}</span>
    </header>
    <div class="vdp-datepicker__body vdp-datepicker__body_picker_month">
      <span
        class="cell month"
        v-for="month in months"
        :key="month.timestamp"
        :class="{'selected': month.isSelected, 'disabled': month.isDisabled}"
        @click.stop="selectMonth(month)"
      >
        <span class="cell__inner">{{ month.month }}</span>
      </span>
    </div>
  </div>
</template>
<script>
import { makeDateUtils } from '../utils/DateUtils'
export default {
  props: {
    showDateHeader: Boolean,
    showMonthView: Boolean,
    selectedDate: Date,
    pageDate: Date,
    pageTimestamp: Number,
    disabledDates: Object,
    calendarClass: [String, Object, Array],
    calendarStyle: Object,
    translation: Object,
    isRtl: Boolean,
    allowedToShowView: Function,
    useUtc: Boolean
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      utils: constructedDateUtils
    }
  },
  computed: {
    months () {
      const d = this.pageDate
      let months = []
      // set up a new date object to the beginning of the current 'page'
      let dObj = this.useUtc
        ? new Date(Date.UTC(d.getUTCFullYear(), 0, d.getUTCDate()))
        : new Date(
            d.getFullYear(),
            0,
            d.getDate(),
            d.getHours(),
            d.getMinutes()
          )
      for (let i = 0; i < 12; i++) {
        months.push({
          month: this.utils.getMonthName(i, this.translation.months),
          timestamp: dObj.getTime(),
          isSelected: this.isSelectedMonth(dObj),
          isDisabled: this.isDisabledMonth(dObj)
        })
        this.utils.setMonth(dObj, this.utils.getMonth(dObj) + 1)
      }
      return months
    },
    currDay () {
      if (this.selectedDate) {
        return this.utils.getDate(this.selectedDate)
      }

      return 1
    },
    /**
     * Gets the name of the month the current page is on
     * @return {String}
     */
    currMonthName () {
      return this.utils.getMonthName(
        this.utils.getMonth(this.pageDate),
        this.translation.months
      )
    },
    /**
     * Gets the name of the year that current page is on
     * @return {Number}
     */
    currYearName () {
      const yearSuffix = this.translation.yearSuffix
      return `${this.utils.getFullYear(this.pageDate)}${yearSuffix}`
    },
    /**
     * Is the left hand navigation disabled
     * @return {Boolean}
     */
    isLeftNavDisabled () {
      return this.isRtl
        ? this.isNextYearDisabled(this.pageTimestamp)
        : this.isPreviousYearDisabled(this.pageTimestamp)
    },
    /**
     * Is the right hand navigation disabled
     * @return {Boolean}
     */
    isRightNavDisabled () {
      return this.isRtl
        ? this.isPreviousYearDisabled(this.pageTimestamp)
        : this.isNextYearDisabled(this.pageTimestamp)
    }
  },
  methods: {
    /**
     * Emits a selectMonth event
     * @param {Object} month
     */
    selectMonth (month) {
      if (month.isDisabled) {
        return false
      }
      this.$emit('selectMonth', month)
    },
    /**
     * Changes the year up or down
     * @param {Number} incrementBy
     */
    changeYear (incrementBy) {
      let date = this.pageDate
      this.utils.setFullYear(date, this.utils.getFullYear(date) + incrementBy)
      this.$emit('changedYear', date)
    },
    /**
     * Decrements the year
     */
    previousYear () {
      if (!this.isPreviousYearDisabled()) {
        this.changeYear(-1)
      }
    },
    /**
     * Checks if the previous year is disabled or not
     * @return {Boolean}
     */
    isPreviousYearDisabled () {
      if (!this.disabledDates || !this.disabledDates.to) {
        return false
      }
      return (
        this.utils.getFullYear(this.disabledDates.to) >=
        this.utils.getFullYear(this.pageDate)
      )
    },
    /**
     * Increments the year
     */
    nextYear () {
      if (!this.isNextYearDisabled()) {
        this.changeYear(1)
      }
    },
    /**
     * Checks if the next year is disabled or not
     * @return {Boolean}
     */
    isNextYearDisabled () {
      if (!this.disabledDates || !this.disabledDates.from) {
        return false
      }
      return (
        this.utils.getFullYear(this.disabledDates.from) <=
        this.utils.getFullYear(this.pageDate)
      )
    },
    showDayCalendar () {
      this.$emit('showDayCalendar')
    },
    /**
     * Emits an event that shows the year calendar
     */
    showYearCalendar () {
      this.$emit('showYearCalendar')
    },
    /**
     * Whether the selected date is in this month
     * @param {Date}
     * @return {Boolean}
     */
    isSelectedMonth (date) {
      return (
        this.selectedDate &&
        this.utils.getFullYear(this.selectedDate) ===
          this.utils.getFullYear(date) &&
        this.utils.getMonth(this.selectedDate) === this.utils.getMonth(date)
      )
    },
    /**
     * Whether a month is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isDisabledMonth (date) {
      let disabledDates = false

      if (typeof this.disabledDates === 'undefined') {
        return false
      }

      if (
        typeof this.disabledDates.to !== 'undefined' &&
        this.disabledDates.to
      ) {
        if (
          (this.utils.getMonth(date) <
            this.utils.getMonth(this.disabledDates.to) &&
            this.utils.getFullYear(date) <=
              this.utils.getFullYear(this.disabledDates.to)) ||
          this.utils.getFullYear(date) <
            this.utils.getFullYear(this.disabledDates.to)
        ) {
          disabledDates = true
        }
      }
      if (
        typeof this.disabledDates.from !== 'undefined' &&
        this.disabledDates.from
      ) {
        if (
          (this.utils.getMonth(date) >
            this.utils.getMonth(this.disabledDates.from) &&
            this.utils.getFullYear(date) >=
              this.utils.getFullYear(this.disabledDates.from)) ||
          this.utils.getFullYear(date) >
            this.utils.getFullYear(this.disabledDates.from)
        ) {
          disabledDates = true
        }
      }

      if (
        typeof this.disabledDates.customPredictor === 'function' &&
        this.disabledDates.customPredictor(date)
      ) {
        disabledDates = true
      }
      return disabledDates
    }
  }
}
// eslint-disable-next-line
</script>
