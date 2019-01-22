<template>
  <div class="c-datepicker__calendar">
    <div class="c-month-selector">
      <div
        class="c-month-selector__prev-month"
        @click="prevMonth">
        <a v-if="!isCurrentMonthOrBelow"><span class="screen-reader-text">Previous Month</span></a>
      </div>
      <div class="c-month-selector__current-month">
        <span class="c-month">{{ monthName }}</span>
        <span class="c-year">{{ year }}</span>
      </div>
      <div
        class="c-month-selector__next-month"
        @click="nextMonth">
        <a><span class="screen-reader-text">Next Month</span></a>
      </div>
    </div>
    <table class="c-calendar">
      <thead>
        <tr class="c-calendar__weekdays">
          <th class="c-calendar__name"><span>Mo</span></th>
          <th class="c-calendar__name"><span>Tu</span></th>
          <th class="c-calendar__name"><span>We</span></th>
          <th class="c-calendar__name"><span>Th</span></th>
          <th class="c-calendar__name"><span>Fr</span></th>
          <th class="c-calendar__name"><span>Sa</span></th>
          <th class="c-calendar__name"><span>Su</span></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="week in calendarWeeks">
          <td
            v-for="day in week"
            :class="day.class"
            class="c-calendar__day"
            @click="day.class['selected-timespan'] && selectDate({
              year: year,
              month: monthNumber,
              day: day.display,
            })"
          >
            <span>{{ day.display }}</span>
          </td>
        </tr>
      </tbody>
    </table>

  </div>
</template>

<script>
import config from '../../../../config'

export default {
  props: {
    date: {
      type: Object,
      required: true,
    },
  },

  data () {
    return {
      now: new Date(),
      time: new Date(),
    }
  },

  computed: {
    isCurrentMonthOrBelow () {
      return this.now.getMonth() >= this.time.getMonth() &&
        this.now.getFullYear() >= this.time.getFullYear()
    },

    monthIndex () {
      return this.time.getMonth()
    },

    monthName () {
      return config.monthNames[this.monthIndex]
    },

    monthNumber () {
      // Convert index to 1 based month number
      return this.monthIndex + 1
    },

    year () {
      return this.time.getFullYear()
    },

    calendarWeeks () {
      const weeks = []
      const daysInMonth = new Date(this.year, this.monthIndex + 1, 0).getDate()
      const shortDayNameOfMonthStart = String(this.time).split(' ').slice(0)[0]
      const startDayWeekIndex = config.dayName.findIndex(day => day.slice(0, 3) === shortDayNameOfMonthStart)
      const formatDay = dayIndex => (dayIndex + 1).toString().padStart(2, '0')
      const dayClasses = (day, weekIndex) => ({
        'selected-timespan': this.isActive(day, weekIndex),
        'current-date': this.isToday(day),
        'end-date': this.isSelected(day, weekIndex),
      })
      let currentWeek

      // Populate currrent month
      for (let i = startDayWeekIndex; i < startDayWeekIndex + daysInMonth; i++) {
        const day = formatDay(i - startDayWeekIndex)

        if (i === startDayWeekIndex || i % 7 === 0) {
          currentWeek = Array.from(Array(7)).fill('')
          weeks.push(currentWeek)
        }

        currentWeek[i % 7] = {
          display: day,
          class: dayClasses(day, weeks.length - 1),
        }
      }

      // Fill last week with next month dates
      const newMonthIndex = currentWeek.indexOf('')
      if (newMonthIndex > -1) {
        for (let i = newMonthIndex; i < 7; i++) {
          const day = formatDay(i - newMonthIndex)
          currentWeek[i] = {
            display: day,
            class: dayClasses(day, weeks.length - 1),
          }
        }
      }

      return weeks
    },
  },

  created () {
    this.setTime([this.year, this.monthNumber])
    if (!this.date.day) {
      this.selectDate({
        year: this.year,
        month: this.monthNumber,
        day: this.now.getDate() + 1,
      })
    }
  },

  methods: {
    setTime (arr = []) {
      this.time = new Date(arr)
    },

    prevMonth () {
      if (this.isCurrentMonthOrBelow) {
        return
      }
      const year = this.monthNumber === 1 ? this.year - 1 : this.year
      const month = this.monthNumber === 1 ? 12 : this.monthNumber - 1
      this.setTime([year, month])
    },

    nextMonth () {
      const year = this.monthNumber === 12 ? this.year + 1 : this.year
      const month = (this.monthNumber % 12) + 1
      this.setTime([year, month])
    },

    selectDate (date) {
      this.$emit('selectDate', date)
    },

    isActive (day, weekIndex) {
      const isTodayOrAfter = parseInt(day) >= this.now.getDate()
      const isNextMonthDays = weekIndex > 2 && parseInt(day) <= 7
      const isCurrentMonth = this.monthIndex === this.now.getMonth()
      const isNextMonth = (this.monthIndex > this.now.getMonth() && this.year === this.now.getFullYear()) ||
        this.year > this.now.getFullYear()

      return (isTodayOrAfter && isCurrentMonth) ||
        (isNextMonth && !isNextMonthDays)
    },

    isToday (day) {
      return parseInt(day) === this.now.getDate() &&
        this.monthIndex === this.now.getMonth()
    },

    isSelected (day, weekIndex) {
      return parseInt(this.date.day) === parseInt(day) &&
        this.date.month === this.monthNumber &&
        this.isActive(day, weekIndex) &&
        this.date.year === this.year
    },
  },
}
</script>
