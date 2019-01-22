<template>
  <section class="c-connection-modal c-connection-modal--datepicker">
    <header class="c-modal-header">
      <p class="c-modal-description">Select a Day and Time that your order will be good until.</p>
    </header>
    <div class="c-modal-content">
      <div class="c-modal-content__column c-modal-content__column--calendar">
        <DatePickerCalendar
          :date="selectedDateTime"
          @selectDate="handleSelectDateTime" />
      </div>
      <div class="c-modal-content__column c-modal-content__column--time">
        <DatePickerTime
          :time="selectedDateTime"
          @selectTime="handleSelectDateTime" />
      </div>
      <div class="c-step-footer">
        <button
          class="c-confirm-button c-small-button c-small-button--blue"
          @click="handleConfirm">
          Confirm
        </button>
      </div>
    </div>
  </section>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import DatePickerCalendar from './calendar'
import DatePickerTime from './time'

export default {

  components: {
    DatePickerCalendar,
    DatePickerTime,
  },

  props: {
    vuexModule: {
      type: String,
      required: true,
    },
  },

  data () {
    return {
      selectedDateTime: {},
    }
  },

  computed: {
    ...mapState({
      datetime (state, getters) {
        return getters[this.vuexModule + '/datetime']
      },
    }),
  },

  created () {
    this.selectedDateTime = this.datetime
  },

  methods: {
    ...mapActions({
      setDatetime (dispatch, payload) {
        return dispatch(this.vuexModule + '/setDatetime', payload)
      },
    }),

    handleSelectDateTime (datetime) {
      this.selectedDateTime = {
        ...this.selectedDateTime,
        ...datetime,
      }
    },

    handleConfirm () {
      this.setDatetime(this.selectedDateTime)
      this.$root.$emit('closeModal')
    },
  },
}
</script>
