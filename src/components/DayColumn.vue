<template>
  <td
    v-for="minute in getDayTimeSlots()"
    :key="minute"
    :class="getTimeClass(minute)"
    :data-day="day"
    :data-minute="minute"
    @mouseenter="$emit('cell-enter', minute)"
  ></td>
</template>

<script>
export default {
  props: {
    day: {
      type: String,
      required: true,
    },
    schedule: {
      type: Object,
      required: true,
    },
    selectedMinutes: {
      type: Set,
      required: true,
    },
  },
  methods: {
    getDayTimeSlots() {
      const slots = [];
      for (let i = 0; i < 1440; i += 60) {
        slots.push(i);
      }
      return slots;
    },
    getTimeClass(minute) {
      const intervals = this.schedule[this.day] || [];
      const isInSchedule = intervals.some(
        (interval) => minute >= interval.bt && minute < interval.et
      );
      return this.selectedMinutes.has(minute) || isInSchedule ? "busy" : "";
    },
  },
};
</script>

<style scoped>
td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
}
.busy {
  background-color: #e2e2e2;
}
</style>
