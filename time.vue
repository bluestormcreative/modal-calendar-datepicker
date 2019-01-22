<template>
  <div class="c-datepicker__time">
    <div class="c-time-inputs">
      <div class="c-time">
        <span
          :class="{'active': isHoursMode}"
          class="c-time__hour"
          @click="setHoursMode(true)">
          {{ displayHours }} :
        </span>
        <span
          :class="{'active': !isHoursMode}"
          class="c-time__minute"
          @click="setHoursMode(false)">
          {{ displayMinutes }}
        </span>
        <span class="c-time_time-of-day">{{ dayTime }}</span>
      </div>
      <div class="c-timezone">
        <span>{{ timezone }}</span>
      </div>
    </div>
    <div class="c-timepicker">
      <div class="c-timepicker__time-of-day">
        <span
          :class="dayTimeClass('AM')"
          @click="setDayTime('AM')">
          AM
        </span>
        <span
          :class="dayTimeClass('PM')"
          @click="setDayTime('PM')">
          PM
        </span>
      </div>
      <div class="c-timepicker__clock">
        <div
          :style="rotateSelector"
          :class="{'minutes': !isHoursMode}"
          class="c-time-selector">
          <div class="c-selector-mask" />
        </div>
        <ul class="c-digits">
          <li
            v-for="interval in clockIntervals"
            @click="setClockTime(interval)">
            {{ interval }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    time: {
      type: Object,
      required: true,
    },
  },

  data () {
    return {
      now: new Date(),
      dayTime: '',
      isHoursMode: true,
    }
  },

  computed: {
    clockIntervals () {
      return this.isHoursMode
        ? Array.from(Array(12), (_, i) => (i + 1))
        : Array.from(Array(12), (_, i) => (i === 11)
          ? 0
          : (i + 1) * 5
        )
    },

    timezone () {
      const gmtZeroDistance = new Date().getTimezoneOffset() / 60
      const gmtOffset = (gmtZeroDistance > 0 ? '-' : '+') + Math.abs(gmtZeroDistance)

      return `GMT${gmtOffset}`
    },

    displayHours () {
      const hour = this.time.hour % 12 === 0
        ? 12
        : this.time.hour % 12

      return String(hour).padStart(2, '0')
    },

    displayMinutes () {
      return String(this.time.minutes).padStart(2, '0')
    },

    rotateSelector () {
      const one = 360 / 12
      const n = this.isHoursMode
        ? one * this.time.hour
        : one * (this.time.minutes / 5)
      return `transform: rotate(${n}deg)`
    },
  },

  created () {
    if (!this.time.hour) {
      this.$emit('selectTime', {
        hour: this.now.getHours(),
        minutes: this.now.getMinutes(),
      })
    }

    const hours = this.time.hour ? this.time.hour : this.now.getHours()
    this.dayTime = hours < 12 ? 'AM' : 'PM'
  },

  methods: {
    setHoursMode (mode) {
      this.isHoursMode = mode
    },

    dayTimeClass (dayTime) {
      return this.dayTime === dayTime
        ? 'selected'
        : ''
    },

    setClockTime (time) {
      const hour = {
        hour: this.dayTime === 'PM' ? time + 12 : time,
      }
      const minutes = {
        minutes: time,
      }
      const selectedTime = this.isHoursMode
        ? hour
        : minutes

      this.$emit('selectTime', selectedTime)
    },

    setDayTime (dayTime) {
      this.dayTime = dayTime
    },
  },
}
</script>
