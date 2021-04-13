<template>
  <transition name="asd__fade">
    <!-- eslint-disable -->
    <div
      :id="wrapperId"
      class="asd__wrapper"
      v-show="showDatepicker"
      :class="wrapperClasses"
      :style="showFullscreen ? undefined : wrapperStyles"
      v-click-outside="handleClickOutside"
    >
      <!-- . -->
      <div class="asd__mobile-header asd__mobile-only" v-if="showFullscreen">
        <button
          type="button"
          class="asd__mobile-close"
          @click="closeDatepicker"
          :aria-label="ariaLabels.closeDatepicker"
        >
          <slot v-if="$slots['close-icon']" name="close-icon"></slot>
          <div v-else class="asd__mobile-close-icon" aria-hidden="true">X</div>
        </button>
        <h3>{{ mobileHeader || mobileHeaderFallback }}</h3>
      </div>

      <div class="asd__datepicker-header">
        <div class="asd__change-month-button asd__change-month-button--previous">
          <button @click="previousMonth" type="button" :aria-label="ariaLabels.previousMonth">
            <slot v-if="$slots['previous-month-icon']" name="previous-month-icon"></slot>
            <svg v-else viewBox="0 0 1000 1000">
              <path
                d="M336.2 274.5l-210.1 210h805.4c13 0 23 10 23 23s-10 23-23 23H126.1l210.1 210.1c11 11 11 21 0 32-5 5-10 7-16 7s-11-2-16-7l-249.1-249c-11-11-11-21 0-32l249.1-249.1c21-21.1 53 10.9 32 32z"
              ></path>
            </svg>
          </button>
        </div>
        <div class="asd__change-month-button asd__change-month-button--next">
          <button @click="nextMonth" type="button" :aria-label="ariaLabels.nextMonth">
            <slot v-if="$slots['next-month-icon']" name="next-month-icon"></slot>
            <svg v-else viewBox="0 0 1000 1000">
              <path
                d="M694.4 242.4l249.1 249.1c11 11 11 21 0 32L694.4 772.7c-5 5-10 7-16 7s-11-2-16-7c-11-11-11-21 0-32l210.1-210.1H67.1c-13 0-23-10-23-23s10-23 23-23h805.4L662.4 274.5c-21-21.1 11-53.1 32-32.1z"
              ></path>
            </svg>
          </button>
        </div>

        <div
          class="asd__days-legend"
          v-for="(month, index) in showMonths"
          :key="month"
          :style="[monthWidthStyles, { left: width * index + 'px' }]"
        >
          <div class="asd__day-title" v-for="(day, index) in daysShort" :key="index">{{ day }}</div>
        </div>
      </div>

      <div class="asd__inner-wrapper" :style="innerStyles">
        <transition-group name="asd__list-complete" tag="div">
          <div
            v-for="(month, monthIndex) in months"
            :key="month.firstDateOfMonth"
            class="asd__month"
            :class="{ 'asd__month--hidden': monthIndex === 0 || monthIndex > showMonths }"
            :style="monthWidthStyles"
          >
            <div class="asd__month-name">
              <select
                v-if="showMonthYearSelect"
                v-model="month.monthName"
                class="asd__month-year-select"
                :tabindex="monthIndex === 0 || monthIndex > showMonths ? -1 : 0"
                @change="updateMonth(monthIndex, month.year, $event)"
                v-resize-select
              >
                <option
                  v-for="(monthName, idx) in monthNames"
                  :value="monthName"
                  :disabled="isMonthDisabled(month.year, idx)"
                  :key="`month-${monthIndex}-${monthName}`"
                >
                  {{ monthName }}
                </option>
              </select>
              <span v-else>{{ month.monthName }}</span>

              <select
                v-if="showMonthYearSelect"
                class="asd__month-year-select"
                :tabindex="monthIndex === 0 || monthIndex > showMonths ? -1 : 0"
                v-model="month.year"
                @change="updateYear(monthIndex, month.monthNumber - 1, $event)"
              >
                <option
                  v-if="years.indexOf(month.year) === -1"
                  :value="month.year"
                  :key="`month-${monthIndex}-${month.year}`"
                  :disabled="true"
                >
                  {{ month.year }}
                </option>
                <!-- <option v-for="year in years" :value="year" :key="`month-${monthIndex}-${year}`">
                  {{ year }}
                </option> -->
              </select>
              <span v-else>{{ month.year }}</span>
            </div>

            <table class="asd__month-table" role="presentation">
              <tbody>
                <tr class="asd__week" v-for="(week, index) in month.weeks" :key="index">
                  <td
                    class="asd__day"
                    v-for="({ fullDate, dayNumber }, index) in week"
                    :key="index + '_' + dayNumber"
                    :data-date="fullDate"
                    :ref="`date-${fullDate}`"
                    :tabindex="
                      isDateVisible(fullDate) && isSameDate(focusedDate, fullDate) ? 0 : -1
                    "
                    :aria-label="isDateVisible(fullDate) ? getAriaLabelForDate(fullDate) : false"
                    :class="[
                      {
                        'asd__day--enabled': dayNumber !== 0,
                        'asd__day--empty': dayNumber === 0,
                        'asd__day--disabled': isDisabled(fullDate),
                        'asd__day--selected':
                          fullDate && (selectedDate1 === fullDate || selectedDate2 === fullDate),
                        'asd__day--in-range': isInRange(fullDate),
                        'asd__day--today': fullDate && isToday(fullDate),
                        'asd__day--hovered': isHoveredInRange(fullDate),
                        'asd__selected-date-one': fullDate && fullDate === selectedDate1,
                        'asd__selected-date-two': fullDate && fullDate === selectedDate2,
                      },
                      customizedDateClass(fullDate),
                    ]"
                    :style="getDayStyles(fullDate)"
                    @mouseover="
                      () => {
                        setHoverDate(fullDate)
                      }
                    "
                  >
                    <button
                      class="asd__day-button"
                      type="button"
                      v-if="dayNumber"
                      tabindex="-1"
                      :date="fullDate"
                      :disabled="isDisabled(fullDate)"
                      @click="
                        () => {
                          selectDate(fullDate)
                        }
                      "
                    >
                      {{ dayNumber }}
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </transition-group>
        <div
          v-if="showShortcutsMenuTrigger"
          :class="{
            'asd__keyboard-shortcuts-menu': true,
            'asd__keyboard-shortcuts-show': showKeyboardShortcutsMenu,
          }"
          :style="keyboardShortcutsMenuStyles"
        >
          <div class="asd__keyboard-shortcuts-title">{{ texts.keyboardShortcuts }}</div>
          <button
            class="asd__keyboard-shortcuts-close"
            ref="keyboard-shortcus-menu-close"
            tabindex="0"
            @click="closeKeyboardShortcutsMenu"
            :aria-label="ariaLabels.closeKeyboardShortcutsMenu"
          >
            <slot v-if="$slots['close-shortcuts-icon']" name="close-shortcuts-icon"></slot>
            <div v-else class="asd__mobile-close-icon" aria-hidden="true">X</div>
          </button>
          <ul class="asd__keyboard-shortcuts-list">
            <li v-for="(shortcut, i) in keyboardShortcuts" :key="i">
              <span
                class="asd__keyboard-shortcuts-symbol"
                :aria-label="shortcut.symbolDescription"
                >{{ shortcut.symbol }}</span
              >
              {{ shortcut.label }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
/* eslint-disable */
import format from 'date-fns/format'
import subMonths from 'date-fns/subMonths'
import addMonths from 'date-fns/addMonths'
import getDaysInMonth from 'date-fns/getDaysInMonth'
import lastDayOfMonth from 'date-fns/lastDayOfMonth'
import getMonth from 'date-fns/getMonth'
import setMonth from 'date-fns/setMonth'
import getYear from 'date-fns/getYear'
import setYear from 'date-fns/setYear'
import isSameMonth from 'date-fns/isSameMonth'
import isSameDay from 'date-fns/isSameDay'
import addDays from 'date-fns/addDays'
import subDays from 'date-fns/subDays'
import addWeeks from 'date-fns/addWeeks'
import subWeeks from 'date-fns/subWeeks'
import startOfMonth from 'date-fns/startOfMonth'
import startOfWeek from 'date-fns/startOfWeek'
import endOfWeek from 'date-fns/endOfWeek'
import isBefore from 'date-fns/isBefore'
import isAfter from 'date-fns/isAfter'
import isValid from 'date-fns/isValid'
import parseISO from 'date-fns/parseISO'

import { debounce, copyObject, findAncestor, randomString } from './../helpers'
import vClickOutside from 'v-click-outside'
import ResizeSelect from '../directives/ResizeSelect'

export default {
  name: 'AirbnbStyleDatepicker',
  directives: {
    clickOutside: vClickOutside.directive,
    resizeSelect: ResizeSelect,
  },
  props: {
    triggerElementId: { type: String },
    dateOne: { type: Date },
    dateTwo: { type: Date },
    minDate: { type: Date },
    endDate: { type: Date },
    mode: { type: String, default: 'range' },
    offsetY: { type: Number, default: 0 },
    offsetX: { type: Number, default: 0 },
    monthsToShow: { type: Number, default: 2 },
    startOpen: { type: Boolean },
    fullscreenMobile: { type: Boolean },
    inline: { type: Boolean },
    mobileHeader: { type: String },
    disabledDates: { type: Array, default: () => [] },
    enabledDates: { type: Array, default: () => [] },
    customizedDates: { type: Array, default: () => [] },
    showActionButtons: { type: Boolean, default: true },
    showShortcutsMenuTrigger: { type: Boolean, default: true },
    showMonthYearSelect: { type: Boolean, default: false },
    yearsForSelect: { type: Number, default: 10 },
    isTest: {
      type: Boolean,
      default: () => process.env.NODE_ENV === 'test',
    },
    trigger: { type: Boolean, default: false },
    closeAfterSelect: { type: Boolean, default: false },
  },
  data() {
    return {
      wrapperId: 'airbnb-style-datepicker-wrapper-' + randomString(5),
      dateFormat: 'yyyy-MM-dd',
      dateLabelFormat: 'dddd, MMMM d, yyyy',
      showDatepicker: false,
      showKeyboardShortcutsMenu: false,
      showMonths: 2,
      colors: {
        selected: '#00a699',
        inRange: '#66e2da',
        selectedText: '#fff',
        text: '#565a5c',
        inRangeBorder: '#33dacd',
        disabled: '#fff',
        hoveredInRange: '#67f6ee',
      },
      sundayFirst: false,
      ariaLabels: {
        chooseDate: date => date,
        chooseStartDate: date => `Choose ${date} as your start date.`,
        chooseEndDate: date => `Choose ${date} as your end date.`,
        selectedDate: date => `Selected. ${date}`,
        unavailableDate: date => `Not available. ${date}`,
        previousMonth: 'Move backward to switch to the previous month.',
        nextMonth: 'Move forward to switch to the next month.',
        closeDatepicker: 'Close calendar',
        openKeyboardShortcutsMenu: 'Open keyboard shortcuts menu.',
        closeKeyboardShortcutsMenu: 'Close keyboard shortcuts menu',
      },
      monthNames: [
        'January',
        'February',
        'March',
        'April',
        'May',
        'June',
        'July',
        'August',
        'September',
        'October',
        'November',
        'December',
      ],
      days: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'],
      daysShort: ['Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat', 'Sun'],
      texts: {
        apply: 'Apply',
        cancel: 'Cancel',
        keyboardShortcuts: 'Keyboard Shortcuts',
      },
      keyboardShortcuts: [
        { symbol: '↵', label: 'Select the date in focus', symbolDescription: 'Enter key' },
        {
          symbol: '←/→',
          label: 'Move backward (left) and forward (right) by one day.',
          symbolDescription: 'Left or right arrow keys',
        },
        {
          symbol: '↑/↓',
          label: 'Move backward (up) and forward (down) by one week.',
          symbolDescription: 'Up or down arrow keys',
        },
        {
          symbol: 'PgUp/PgDn',
          label: 'Switch months.',
          symbolDescription: 'PageUp and PageDown keys',
        },
        {
          symbol: 'Home/End',
          label: 'Go to the first or last day of a week.',
          symbolDescription: 'Home or End keys',
        },
        { symbol: 'Esc', label: 'Close this panel', symbolDescription: 'Escape key' },
        { symbol: '?', label: 'Open this panel', symbolDescription: 'Question mark' },
      ],
      keys: {
        arrowDown: 40,
        arrowUp: 38,
        arrowRight: 39,
        arrowLeft: 37,
        enter: 13,
        pgUp: 33,
        pgDn: 34,
        end: 35,
        home: 36,
        questionMark: 191,
        esc: 27,
      },
      startingDate: null,
      months: [],
      years: [],
      width: 300,
      selectedDate1: null,
      selectedDate2: null,
      isSelectingDate1: true,
      hoverDate: null,
      focusedDate: null,
      alignRight: false,
      triggerPosition: {},
      triggerWrapperPosition: {},
      viewportWidth: undefined,
      isMobile: undefined,
      isTablet: undefined,
      triggerElement: undefined,
    }
  },
  computed: {
    wrapperClasses() {
      return {
        'asd__wrapper--datepicker-open': this.showDatepicker,
        'asd__wrapper--full-screen': this.showFullscreen,
        'asd__wrapper--inline': this.inline,
      }
    },
    wrapperStyles() {
      return {
        position: this.inline ? 'static' : 'absolute',
        top: this.inline ? '0' : this.triggerPosition.height + this.offsetY + 'px',
        left: !this.alignRight
          ? this.triggerPosition.left - this.triggerWrapperPosition.left + this.offsetX + 'px'
          : '',
        right: this.alignRight
          ? this.triggerWrapperPosition.right - this.triggerPosition.right + this.offsetX + 'px'
          : '',
        width: this.width * this.showMonths + 'px',
        zIndex: this.inline ? '0' : '100',
      }
    },
    innerStyles() {
      return {
        'margin-left': this.showFullscreen ? '-' + this.viewportWidth : `-${this.width}px`,
      }
    },
    keyboardShortcutsMenuStyles() {
      return {
        left: this.showFullscreen ? this.viewportWidth : `${this.width}px`,
      }
    },
    showFullscreen() {
      return this.isMobile && this.fullscreenMobile
    },
    mobileHeaderFallback() {
      return this.mode === 'range' ? 'Select dates' : 'Select date'
    },
    monthWidthStyles() {
      return {
        width: this.showFullscreen ? this.viewportWidth : this.width + 'px',
      }
    },
  },
  watch: {},
  created() {
    this.setupDatepicker()

    if (this.sundayFirst) {
      this.setSundayToFirstDayInWeek()
    }
  },
  mounted() {
    this.viewportWidth = window.innerWidth + 'px'
    this.isMobile = window.innerWidth < 768
    this.isTablet = window.innerWidth >= 768 && window.innerWidth <= 1024
    // this._handleWindowResizeEvent = debounce(() => {
    //   this.positionDatepicker()
    //   this.setStartDates()
    // }, 200)
    this._handleWindowClickEvent = event => {
      if (event.target.id === this.triggerElementId) {
        event.stopPropagation()
        event.preventDefault()
        this.toggleDatepicker()
      }
    }

    // window.addEventListener('resize', this._handleWindowResizeEvent)

    this.triggerElement = this.isTest
      ? document.createElement('input')
      : document.getElementById(this.triggerElementId)

    this.setStartDates()
    this.generateMonths()
    //     this.generateYears()

    if (this.startOpen || this.inline) {
      this.openDatepicker()
    }

    this.$el.addEventListener('keyup', this.handleKeyboardInput)
    this.$el.addEventListener('keydown', this.trapKeyboardInput)
    this.triggerElement.addEventListener('keyup', this.handleTriggerInput)
    this.triggerElement.addEventListener('click', this._handleWindowClickEvent)
  },
  destroyed() {
    window.removeEventListener('resize', this._handleWindowResizeEvent)
    window.removeEventListener('click', this._handleWindowClickEvent)

    this.$el.removeEventListener('keyup', this.handleKeyboardInput)
    this.$el.removeEventListener('keydown', this.trapKeyboardInput)
    this.triggerElement.removeEventListener('keyup', this.handleTriggerInput)
    this.triggerElement.removeEventListener('click', this._handleWindowClickEvent)
  },
  methods: {
    getDayStyles(date) {
      const isSelected = this.isSelected(date)
      const isInRange = this.isInRange(date)
      const isDisabled = this.isDisabled(date)
      const isHoveredInRange = this.isHoveredInRange(date)

      let styles = {
        width: (this.width - 30) / 7 + 'px',
        background: isSelected
          ? this.colors.selected
          : isHoveredInRange
          ? this.colors.hoveredInRange
          : isInRange
          ? this.colors.inRange
          : '',
        color: isSelected
          ? this.colors.selectedText
          : isInRange || isHoveredInRange
          ? this.colors.selectedText
          : this.colors.text,
        border: isSelected
          ? '1px double ' + this.colors.selected
          : (isInRange && this.allDatesSelected) || isHoveredInRange
          ? '1px double ' + this.colors.inRangeBorder
          : '',
      }

      if (isDisabled) {
        styles.background = this.colors.disabled
      }
      return styles
    },
    handleClickOutside(event) {
      console.log('handleClickOutside')
      if (event.target.id === this.triggerElementId || !this.showDatepicker || this.inline) {
        return
      }
      this.closeDatepicker()
    },

    openDatepicker() {
      this.positionDatepicker()
      this.setStartDates()
      this.triggerElement.classList.add('datepicker-open')
      this.showDatepicker = true
      this.initialDate1 = this.dateOne
      this.initialDate2 = this.dateTwo
      console.log('openDatepicker', this.initialDate1, this.initialDate2)
      // console.log();
      this.$emit('opened')
      this.$nextTick(() => {
        if (!this.inline) this.setFocusedDate(this.focusedDate)
      })
    },
    closeDatepicker() {
      console.log('closeDatepicker')
      if (this.inline) {
        return
      }
      this.showDatepicker = false
      this.showKeyboardShortcutsMenu = false
      this.triggerElement.classList.remove('datepicker-open')
      this.$emit('closed')
    },

    closeKeyboardShortcutsMenu() {
      this.showKeyboardShortcutsMenu = false
      this.$nextTick(() => this.setFocusedDate(this.focusedDate))
    },
    isToday(date) {
      return format(new Date(), this.dateFormat) === date
    },
    isSelected(date) {
      if (!date) {
        return
      }
      return this.selectedDate1 === date || this.selectedDate2 === date
    },
    isInRange(date) {
      date = parseISO(date)
      if (!this.allDatesSelected || this.isSingleMode) {
        return false
      }

      return (
        (isAfter(date, this.selectedDate1) && isBefore(date, this.selectedDate2)) ||
        (isAfter(date, this.selectedDate1) &&
          isBefore(date, this.hoverDate) &&
          !this.allDatesSelected)
      )
    },
    isHoveredInRange(date) {
      date = parseISO(date)
      if (this.isSingleMode || this.allDatesSelected) {
        return false
      }

      return (
        (isAfter(date, this.selectedDate1) && isBefore(date, this.hoverDate)) ||
        (isAfter(date, this.hoverDate) && isBefore(date, this.selectedDate1))
      )
    },
    isBeforeMinDate(date) {
      if (!this.minDate) {
        return false
      }
      return isBefore(date, this.minDate)
    },
    isAfterEndDate(date) {
      if (!this.endDate) {
        return false
      }
      return isAfter(date, this.endDate)
    },
    isDateDisabled(date) {
      if (this.enabledDates.length > 0) {
        return this.enabledDates.indexOf(date) === -1
      } else {
        return this.disabledDates.indexOf(date) > -1
      }
    },
    customizedDateClass(date) {
      var customizedClasses = ''
      if (this.customizedDates.length > 0) {
        for (var i = 0; i < this.customizedDates.length; i++) {
          if (this.customizedDates[i].dates.indexOf(date) > -1)
            customizedClasses += ` asd__day--${this.customizedDates[i].cssClass}`
        }
      }
      return customizedClasses
    },
    isDisabled(date) {
      return this.isDateDisabled(date) || this.isBeforeMinDate(date) || this.isAfterEndDate(date)
    },
    previousMonth() {
      console.log('')
      console.log('previousMonth')
      // this.startingDate = this.subtractMonths(this.months[0].firstDateOfMonth)
      // this.months.unshift(this.getMonth(this.startingDate))
      // this.months.splice(this.months.length - 1, 1)
      // this.$emit('previous-month', this.visibleMonths)
      // this.resetFocusedDate(false)
    },
    nextMonth() {
      console.log('nextMonth')
      // this.startingDate = this.addMonths(this.months[this.months.length - 1].firstDateOfMonth)
      // this.months.push(this.getMonth(this.startingDate))
      // this.months.splice(0, 1)
      // this.$emit('next-month', this.visibleMonths)
      // this.resetFocusedDate(true)
    },
    subtractMonths(date) {
      console.log('subtractMonths', date)
      console.log('subtractMonths', subMonths(date, 1))
      console.log('subtractMonths', format(subMonths(date, 1), this.dateFormat))
      return format(subMonths(date, 1), this.dateFormat)
    },
    generateMonths() {
      this.months = []
      let currentMonth = this.startingDate
      console.log('generateMonths', currentMonth)
      for (let i = 0; i < this.showMonths + 2; i++) {
        this.months.push(this.getMonth(currentMonth))
        // currentMonth = this.addMonths(currentMonth)
      }
      console.log('generateMonths', this.months)
    },
    getMonth(date) {
      date = parseISO(date)
      const firstDateOfMonth = format(date, 'yyyy-MM-01')
      const year = format(date, 'yyyy')
      const monthNumber = parseInt(format(date, 'M'))
      const monthName = this.monthNames[monthNumber - 1]

      return {
        year,
        firstDateOfMonth,
        monthName,
        monthNumber,
        weeks: this.getWeeks(firstDateOfMonth),
      }
    },

    getWeeks(date) {
      date = parseISO(date)
      const weekDayNotInMonth = { dayNumber: 0 }
      const daysInMonth = getDaysInMonth(date)
      const year = format(date, 'yyyy')
      const month = format(date, 'MM')
      let firstDayInWeek = parseInt(format(date, 'd'))
      if (this.sundayFirst) {
        firstDayInWeek++
      }
      let weeks = []
      let week = []

      // add empty days to get first day in correct position
      for (let s = 1; s < firstDayInWeek; s++) {
        week.push(weekDayNotInMonth)
      }
      for (let d = 0; d < daysInMonth; d++) {
        let isLastDayInMonth = d >= daysInMonth - 1
        let dayNumber = d + 1
        let dayNumberFull = dayNumber < 10 ? '0' + dayNumber : dayNumber
        week.push({
          dayNumber,
          dayNumberFull: dayNumberFull,
          fullDate: year + '-' + month + '-' + dayNumberFull,
        })

        if (week.length === 7) {
          weeks.push(week)
          week = []
        } else if (isLastDayInMonth) {
          for (let i = 0; i < 7 - week.length; i++) {
            week.push(weekDayNotInMonth)
          }
          weeks.push(week)
          week = []
        }
      }
      return weeks
    },
    toggleDatepicker() {
      if (this.showDatepicker) {
        this.closeDatepicker()
      } else {
        this.openDatepicker()
      }
    },
    setupDatepicker() {
      console.log('setupDatepicker')
      if (this.$options.ariaLabels) {
        this.ariaLabels = copyObject(this.$options.ariaLabels)
      }
      if (this.$options.keyboardShortcuts) {
        this.keyboardShortcuts = copyObject(this.$options.keyboardShortcuts)
      }
      if (this.$options.dateLabelFormat) {
        this.dateLabelFormat = copyObject(this.$options.dateLabelFormat)
      }
      if (this.$options.sundayFirst) {
        this.sundayFirst = copyObject(this.$options.sundayFirst)
      }
      if (this.$options.colors) {
        const colors = copyObject(this.$options.colors)
        this.colors.selected = colors.selected || this.colors.selected
        this.colors.inRange = colors.inRange || this.colors.inRange
        this.colors.hoveredInRange = colors.hoveredInRange || this.colors.hoveredInRange
        this.colors.selectedText = colors.selectedText || this.colors.selectedText
        this.colors.text = colors.text || this.colors.text
        this.colors.inRangeBorder = colors.inRangeBorder || this.colors.inRangeBorder
        this.colors.disabled = colors.disabled || this.colors.disabled
      }
      if (this.$options.monthNames && this.$options.monthNames.length === 12) {
        this.monthNames = copyObject(this.$options.monthNames)
      }
      if (this.$options.days && this.$options.days.length === 7) {
        this.days = copyObject(this.$options.days)
      }
      if (this.$options.daysShort && this.$options.daysShort.length === 7) {
        this.daysShort = copyObject(this.$options.daysShort)
      }
      if (this.$options.texts) {
        const texts = copyObject(this.$options.texts)
        this.texts.apply = texts.apply || this.texts.apply
        this.texts.cancel = texts.cancel || this.texts.cancel
      }
    },
    setStartDates() {
      let startDate = this.dateOne || new Date()
      // .toISOString()
      // startDate = parseISO(startDate)
      if (this.hasMinDate && isBefore(startDate, this.minDate)) {
        startDate = this.minDate
      }
      this.startingDate = this.subtractMonths(startDate)
      this.selectedDate1 = this.dateOne
      this.selectedDate2 = this.dateTwo
      this.focusedDate = startDate
    },
    setSundayToFirstDayInWeek() {
      console.log(setSundayToFirstDayInWeek)
      const lastDay = this.days.pop()
      this.days.unshift(lastDay)
      const lastDayShort = this.daysShort.pop()
      this.daysShort.unshift(lastDayShort)
    },
    positionDatepicker() {
      const triggerWrapperElement = findAncestor(this.triggerElement, '.datepicker-trigger')
      this.triggerPosition = this.triggerElement.getBoundingClientRect()
      if (triggerWrapperElement) {
        this.triggerWrapperPosition = triggerWrapperElement.getBoundingClientRect()
      } else {
        this.triggerWrapperPosition = { left: 0, right: 0 }
      }

      const viewportWidth = document.documentElement.clientWidth || window.innerWidth
      this.viewportWidth = viewportWidth + 'px'
      this.isMobile = viewportWidth < 768
      this.isTablet = viewportWidth >= 768 && viewportWidth <= 1024
      this.showMonths = this.isMobile
        ? 1
        : this.isTablet && this.monthsToShow > 2
        ? 2
        : this.monthsToShow

      this.$nextTick(function () {
        const datepickerWrapper = document.getElementById(this.wrapperId)
        if (!this.triggerElement || !datepickerWrapper) {
          return
        }

        const rightPosition =
          this.triggerElement.getBoundingClientRect().left +
          datepickerWrapper.getBoundingClientRect().width
        this.alignRight = rightPosition > viewportWidth
      })
    },

    // setStyles
    // setFocusedDate(date) {
    //   // const formattedDate = format(date, this.dateFormat)
    //   // this.focusedDate = formattedDate
    //   // const dateElement = this.$refs[`date-${formattedDate}`]
    //   // // handle .focus() on ie11 by adding a short timeout
    //   // if (dateElement && dateElement.length) {
    //   //   setTimeout(function () {
    //   //     dateElement[0].focus()
    //   //   }, 10)
    //   // }
    // },
  },
}
</script>

<style lang="scss">
@import './../styles/transitions';

$tablet: 768px;
$color-gray: rgba(0, 0, 0, 0.2);
$border-normal: 1px solid $color-gray;
$border: 1px solid #e4e7e7;
$transition-time: 0.3s;

.datepicker-trigger {
  position: relative;
  overflow: visible;
}

.asd {
  &__wrapper {
    border: $border-normal;
    white-space: nowrap;
    text-align: center;
    overflow: hidden;
    background-color: white;

    *,
    *:after,
    *:before {
      box-sizing: border-box;
    }

    &--full-screen {
      position: fixed;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      border: none;
      z-index: 100;
    }
  }
  &__inner-wrapper {
    transition: all $transition-time ease;
    position: relative;
  }
  &__datepicker-header {
    position: relative;
  }
  &__keyboard-shortcuts-trigger-wrapper {
    position: relative;
  }
  &__keyboard-shortcuts-trigger {
    background-color: transparent;
    cursor: pointer;
    position: absolute;
    bottom: 0px;
    right: 0px;
    font: inherit;
    border-width: 26px 33px 0px 0px;
    border-top: 26px solid transparent;
    border-right: 33px solid rgb(0, 166, 153);

    span {
      color: rgb(255, 255, 255);
      position: absolute;
      bottom: 0px;
      right: -28px;
    }
  }
  &__keyboard-shortcuts-show {
    display: block !important;
  }
  &__keyboard-shortcuts-close {
    background-color: transparent;
    border: none;
    position: absolute;
    top: 7px;
    right: 5px;
    padding: 5px;
    z-index: 100;
    cursor: pointer;
  }
  &__keyboard-shortcuts-menu {
    display: none;
    position: absolute;
    top: 0px;
    bottom: 0px;
    right: 0px;
    z-index: 10;
    overflow: auto;
    background: rgb(255, 255, 255);
    border-width: 1px;
    border-style: solid;
    border-color: rgb(219, 219, 219);
    border-image: initial;
    border-radius: 2px;
    padding: 22px;
    margin: 33px;
    text-align: left;
  }
  &__keyboard-shortcuts-title {
    font-size: 16px;
    font-weight: bold;
    margin: 0px;
  }
  &__keyboard-shortcuts-list {
    list-style: none;
    margin: 6px 0px;
    padding: 0px;
    white-space: initial;
  }
  &__keyboard-shortcuts-symbol {
    font-family: monospace;
    font-size: 12px;
    text-transform: uppercase;
    background: rgb(242, 242, 242);
    padding: 2px 6px;
    margin-right: 4px;
  }
  &__change-month-button {
    position: absolute;
    top: 12px;
    z-index: 10;
    background: white;

    &--previous {
      left: 0;
      padding-left: 15px;
    }
    &--next {
      right: 0;
      padding-right: 15px;
    }

    > button {
      background-color: white;
      border: $border;
      border-radius: 3px;
      padding: 4px 8px;
      cursor: pointer;

      &:hover {
        border: 1px solid #c4c4c4;
      }

      > svg {
        height: 19px;
        width: 19px;
        fill: #82888a;
      }
    }
  }

  &__days-legend {
    position: absolute;
    top: 50px;
    left: 10px;
    padding: 0 10px;
  }
  &__day-title {
    display: inline-block;
    width: percentage(1/7);
    text-align: center;
    margin-bottom: 4px;
    color: rgba(0, 0, 0, 0.7);
    font-size: 0.8em;
    margin-left: -1px;
  }

  &__month-table {
    border-collapse: collapse;
    border-spacing: 0;
    background: white;
    width: 100%;
    max-width: 100%;
  }

  &__month {
    transition: all $transition-time ease;
    display: inline-block;
    padding: 15px;

    &--hidden {
      height: 275px;
      visibility: hidden;
    }
  }
  &__month-name {
    font-size: 1.3em;
    text-align: center;
    margin: 0 0 30px;
    line-height: 1.4em;
    font-weight: bold;
  }
  &__month-year-select {
    &::-ms-expand {
      display: none;
    }
    -webkit-appearance: none;
    border: none;
    background-color: inherit;
    cursor: pointer;
    color: blue;
    font-size: inherit;
    font-weight: inherit;
    padding: 0;
  }

  &__day {
    $size: 38px;
    line-height: $size;
    height: $size;
    padding: 0;
    overflow: hidden;
    &--enabled {
      border: $border;
      &:hover {
        background-color: #e4e7e7;
      }
      &:focus {
        outline: auto 5px Highlight;
        outline: auto 5px -webkit-focus-ring-color;
      }
    }
    &--disabled,
    &--empty {
      opacity: 0.5;

      button {
        cursor: default;
      }
    }
    &--empty {
      border: none;
    }
    &--disabled {
      &:hover {
        background-color: transparent;
      }
    }
  }
  &__day-button {
    background: transparent;
    width: 100%;
    height: 100%;
    border: none;
    cursor: pointer;
    color: inherit;
    text-align: center;
    user-select: none;
    font-size: 15px;
    font-weight: inherit;
    padding: 0;
  }

  &__action-buttons {
    min-height: 50px;
    padding-top: 10px;
    margin-bottom: 12px;
    button {
      display: block;
      position: relative;
      background: transparent;
      border: none;
      font-weight: bold;
      font-size: 15px;
      cursor: pointer;

      &:hover {
        text-decoration: underline;
      }
      &:nth-child(1) {
        float: left;
        left: 15px;
      }
      &:nth-child(2) {
        float: right;
        right: 15px;
      }
    }
  }

  &__mobile-header {
    border-bottom: $border-normal;
    position: relative;
    padding: 15px 15px 15px 15px !important;
    text-align: center;
    height: 50px;
    h3 {
      font-size: 20px;
      margin: 0;
    }
  }
  &__mobile-only {
    display: none;
    @media (max-width: 600px) {
      display: block;
    }
  }
  &__mobile-close {
    border: none;
    position: absolute;
    top: 7px;
    right: 5px;
    padding: 5px;
    z-index: 100;
    cursor: pointer;

    &__icon {
      position: relative;
      font-size: 1.6em;
      font-weight: bold;
      padding: 0;
    }
  }
}
</style>
