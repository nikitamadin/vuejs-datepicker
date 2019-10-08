<template>
  <div
    :class="[calendarClass, 'vdp-datepicker__calendar']"
    v-show="showYearView"
    :style="calendarStyle"
    @mousedown.prevent
  >
    <slot name="beforeCalendarHeader"></slot>
    <header class="vdp-datepicker__header vdp-datepicker__header_full-date" v-if="showDateHeader">
      <span class="vdp-datepicker__control" @click="showDayCalendar">{{ currDay }}</span>
      <span class="vdp-datepicker__control" @click="showMonthCalendar">{{ currMonthName }}</span>
      <span class="vdp-datepicker__control vdp-datepicker__control_active">{{ currYearName }}</span>
    </header>
    <header class="vdp-datepicker__header" v-else>
      <span
        @click="isRtl ? previousDecade() : nextDecade()"
        class="prev"
        :class="{'disabled': isRightNavDisabled}"
      >&gt;</span>

      <span
        @click="isRtl ? nextDecade() : previousDecade()"
        class="next"
        :class="{'disabled': isLeftNavDisabled}"
      >&lt;</span>
    </header>

    <div class="vdp-datepicker__body vdp-datepicker__body_picker_year">
      <span
        class="cell year"
        v-for="year in years"
        v-show="!year.isDisabled"
        :key="year.timestamp"
        :class="{ 'selected': year.isSelected, 'disabled': year.isDisabled }"
        @click.stop="selectYear(year)"
      >{{ year.year }}</span>
    </div>
  </div>
</template>
<script>
import { makeDateUtils } from '../utils/DateUtils'
export default {
  props: {
    showDateHeader: Boolean,
    showYearView: Boolean,
    selectedDate: Date,
    pageDate: Date,
    pageTimestamp: Number,
    disabledDates: Object,
    highlighted: Object,
    calendarClass: [String, Object, Array],
    calendarStyle: Object,
    translation: Object,
    isRtl: Boolean,
    allowedToShowView: Function,
    useUtc: Boolean
  },
  computed: {
    years () {
      // const d = this.pageDate
      let years = []
      // set up a new date object to the beginning of the current 'page'7
      // let dObj = this.useUtc
      //   ? new Date(Date.UTC(Math.floor(d.getUTCFullYear() / 10) * 10, d.getUTCMonth(), d.getUTCDate()))
      //   : new Date(Math.floor(d.getFullYear()), d.getMonth(), d.getDate(), d.getHours(), d.getMinutes())
      let dObj = new Date(this.pageDate)
      for (let i = 0; i < 25; i++) {
        years.push({
          year: this.utils.getFullYear(dObj),
          timestamp: dObj.getTime(),
          isSelected: this.isSelectedYear(dObj),
          isDisabled: this.isDisabledYear(dObj)
        })
        this.utils.setFullYear(dObj, this.utils.getFullYear(dObj) - 1)
      }
      return years
    },
    /**
     * Is the left hand navigation button disabled?
     * @return {Boolean}
     */
    isLeftNavDisabled () {
      return this.isRtl
        ? this.isNextDecadeDisabled(this.pageTimestamp)
        : this.isPreviousDecadeDisabled(this.pageTimestamp)
    },
    /**
     * Is the right hand navigation button disabled?
     * @return {Boolean}
     */
    isRightNavDisabled () {
      return this.isRtl
        ? this.isPreviousDecadeDisabled(this.pageTimestamp)
        : this.isNextDecadeDisabled(this.pageTimestamp)
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
    }
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      utils: constructedDateUtils
    }
  },
  methods: {
    showMonthCalendar () {
      this.$emit('showMonthCalendar')
    },
    showDayCalendar () {
      this.$emit('showDayCalendar')
    },
    selectYear (year) {
      if (year.isDisabled) {
        return false
      }
      this.$emit('selectYear', year)
    },
    changeYear (incrementBy) {
      let date = this.pageDate
      this.utils.setFullYear(date, this.utils.getFullYear(date) + incrementBy)
      this.$emit('changedDecade', date)
    },
    previousDecade () {
      if (this.isPreviousDecadeDisabled()) {
        return false
      }
      this.changeYear(-10)
    },
    isPreviousDecadeDisabled () {
      if (!this.disabledDates || !this.disabledDates.to) {
        return false
      }
      const disabledYear = this.utils.getFullYear(this.disabledDates.to)
      const lastYearInPreviousPage =
        Math.floor(this.utils.getFullYear(this.pageDate) / 10) * 10 - 1
      return disabledYear > lastYearInPreviousPage
    },
    nextDecade () {
      if (this.isNextDecadeDisabled()) {
        return false
      }
      this.changeYear(10)
    },
    isNextDecadeDisabled () {
      if (!this.disabledDates || !this.disabledDates.from) {
        return false
      }
      const disabledYear = this.utils.getFullYear(this.disabledDates.from)
      const firstYearInNextPage =
        Math.ceil(this.utils.getFullYear(this.pageDate) / 10) * 10
      return disabledYear < firstYearInNextPage
    },

    /**
     * Whether the selected date is in this year
     * @param {Date}
     * @return {Boolean}
     */
    isSelectedYear (date) {
      return (
        this.selectedDate &&
        this.utils.getFullYear(this.selectedDate) ===
          this.utils.getFullYear(date)
      )
    },
    /**
     * Whether a year is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isDisabledYear (date) {
      let disabledDates = false
      if (typeof this.disabledDates === 'undefined' || !this.disabledDates) {
        return false
      }

      if (
        typeof this.disabledDates.to !== 'undefined' &&
        this.disabledDates.to
      ) {
        if (
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
