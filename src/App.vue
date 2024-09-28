<template>
  <div>
    <calendar
      ref="calendarComponent"
      :days="days"
      :times="times"
      :schedule="schedule"
      @save-changes="saveChanges"
    />
    <div class="controlPanel">
      <Button @click="clearChanges">Clear</Button>
      <Button @click="handleSaveChanges">Save Changes</Button>
    </div>
  </div>
</template>

<script>
import Calendar from "@/components/Calendar.vue";
import Button from "@/views/Button.vue";

export default {
  components: {
    Calendar,
    Button,
  },
  data() {
    return {
      days: ["mo", "tu", "we", "th", "fr", "sa", "su"],
      times: [
        "00:00",
        "03:00",
        "06:00",
        "09:00",
        "12:00",
        "15:00",
        "18:00",
        "21:00",
      ],
      schedule: {
        mo: [],
        tu: [],
        we: [],
        th: [],
        fr: [],
        sa: [],
        su: [],
      },
    };
  },
  methods: {
    handleSaveChanges() {
      this.$refs.calendarComponent.saveChanges();
    },
    saveChanges(newSchedule) {
      this.schedule = newSchedule;
    },
    clearChanges() {
      this.$refs.calendarComponent.clearSelection();
      this.schedule = {
        mo: [],
        tu: [],
        we: [],
        th: [],
        fr: [],
        sa: [],
        su: [],
      };
    },
  },
};
</script>

<style>
.controlPanel {
  display: flex;
  gap: 20px;
  justify-content: flex-end;
}
</style>
