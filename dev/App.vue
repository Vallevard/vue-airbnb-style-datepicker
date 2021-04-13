<template>
  <div class="app" :class="{ 'align-right': alignRight }">
    <h1>Examples</h1>
    <div class="buttons">
      <button @click="toggleDatepickers">Hide datepickers</button>
      <button @click="toggleAlign">Toggle alignment</button>
      <button @click="toggleTrigger">Toggle trigger</button>
    </div>
    <div v-if="showDatepickers">
      <div class="datepicker-container single-with-input">
        <h3>Single datepicker with input</h3>
        <div class="datepicker-trigger">
          <input
            type="text"
            id="datepicker-input-single-trigger"
            :value="formatDates(inputSingleDateOne)"
            placeholder="Select dates"
          />

          <airbnb-style-datepicker
            :trigger-element-id="'datepicker-input-single-trigger'"
            :mode="'single'"
            :date-one="inputSingleDateOne"
            :date-two="inputSingleDateTwo"
            :data-start-open="true"
            @date-one-selected="
              val => {
                inputSingleDateOne = val
              }
            "
            :inline="true"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import format from 'date-fns/format'
import parseISO from 'date-fns/parseISO'

export default {
  data() {
    return {
      dateFormat: 'yyyy-MM-dd', //'d MMM',
      inputDateOne: null,
      inputDateTwo: null,
      inputSingleDateOne: null,
      inputSingleDateTwo: null,
      buttonDateOne: null,
      buttonDateTwo: null,
      inlineDateOne: null,
      withDisabledDatesDateOne: null,
      callbackDateOne: null,
      callbackDateTwo: null,
      sundayFirst: false,
      alignRight: false,
      showDatepickers: true,
      trigger: false,
    }
  },
  computed: {
    disabledDates() {
      return [new Date('2018-12-30'), new Date('2018-12-10'), new Date('2018-12-14')]
    },
  },
  created() {
    setTimeout(() => {
      this.inputDateOne = new Date('2019-01-12')
      // this.inputDateTwo = ''
    }, 5000)
  },
  methods: {
    formatDates(dateOne, dateTwo) {
      let formattedDates = ''
      if (dateOne) {
        formattedDates = format(parseISO(dateOne), this.dateFormat)
      }
      if (dateTwo) {
        formattedDates += ' - ' + format(parseISO(dateTwo), this.dateFormat)
      }
      return formattedDates
    },
    toggleAlign() {
      this.alignRight = !this.alignRight
    },
    toggleDatepickers() {
      this.showDatepickers = !this.showDatepickers
    },
    toggleTrigger() {
      this.trigger = !this.trigger
    },
    applyMethod() {
      console.log('apply')
    },
    openedMethod() {
      console.log('opened')
    },
    closedMethod() {
      console.log('closed')
      this.trigger = false
    },
    cancelledMethod() {
      console.log('cancelled')
    },
    changeMonthMethod(visibleMonths) {
      console.log('change months', visibleMonths)
    },
  },
}
</script>

<style lang="scss">
html,
body {
  min-height: 200vh;
  font-size: 14px;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell,
    Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
  line-height: 18px;
  font-weight: 400;
  -webkit-font-smoothing: antialiased;
  padding: 10px;
}
.app {
  &.align-right {
    text-align: right;
  }
}

h1 {
  font-size: 1.8em;
  line-height: 1.5em;
  text-align: center;
}
.datepicker-container {
  padding: 0 30px 20px;
  border: 1px solid rgba(0, 0, 0, 0.2);
  background: rgba(0, 0, 0, 0.01);
  max-width: 600px;
  margin: 0 auto 30px;
  border-radius: 12px;
}

#datepicker-button-trigger {
  background: #008489;
  border: 1px solid #008489;
  color: white;
  padding: 6px 10px;
  border-radius: 4px;
  font-size: 15px;
  font-weight: bold;
  text-align: center;
  min-width: 200px;
}
input {
  padding: 6px 10px;
  border: 1px solid rgba(0, 0, 0, 0.2);
}
.with-input {
  .datepicker-trigger {
    //padding-right: 40px;
  }
}
.with-button {
  .datepicker-trigger {
    //padding-left: 10px;
  }
}
// .inline-with-input {
//   width: 600px;
//   input {
//     width: 100%;
//   }
// }
.buttons {
  max-width: 500px;
  margin: 0 auto 30px;
  text-align: center;
}
</style>
