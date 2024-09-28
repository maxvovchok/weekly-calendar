<template>
  <div
    class="calendar-container"
    @mousedown="startSelection"
    @mousemove="onMouseMove"
    @mouseup="endSelection"
  >
    <table class="calendar-table">
      <thead>
        <tr>
          <th>Day</th>
          <th>ALL DAY</th>
          <th v-for="(time, index) in times" :key="index" colspan="3">
            {{ time }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(day, index) in days" :key="index">
          <td :class="{ busyDay: isDayOccupied(day) }">{{ day }}</td>
          <td class="allDay">
            <input
              type="checkbox"
              @change="selectAllDay(day)"
              :checked="isAllDaySelected(day)"
            />
          </td>
          <day-column
            :day="day"
            :schedule="schedule"
            :selected-minutes="selectedMinutes[day] || new Set()"
            @cell-enter="onCellEnter(day, $event)"
          />
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import DayColumn from "@/components/DayColumn.vue";

export default {
  components: {
    DayColumn,
  },
  props: {
    days: {
      type: Array,
      required: true,
    },
    times: {
      type: Array,
      required: true,
    },
    schedule: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      isSelecting: false,
      selectionMode: null,
      selectedMinutes: {},
    };
  },
  methods: {
    isDayOccupied(day) {
      return this.schedule[day] && this.schedule[day].length > 0;
    },
    startSelection(event) {
      this.isSelecting = true;
      const { day, minute } = event.target.dataset;
      if (day && minute !== undefined) {
        const isSelected =
          this.selectedMinutes[day] &&
          this.selectedMinutes[day].has(Number(minute));
        this.selectionMode = isSelected ? "remove" : "add";
        this.toggleSelection(day, Number(minute));
      }
    },
    onMouseMove(event) {
      if (this.isSelecting) {
        const { day, minute } = event.target.dataset;
        if (day && minute !== undefined) {
          this.toggleSelection(day, Number(minute));
        }
      }
    },
    endSelection() {
      this.isSelecting = false;
      this.selectionMode = null;
    },
    toggleSelection(day, minute) {
      if (!this.selectedMinutes[day]) {
        this.selectedMinutes[day] = new Set();
      }
      if (this.selectionMode === "add") {
        this.selectedMinutes[day].add(minute);
      } else if (this.selectionMode === "remove") {
        this.selectedMinutes[day].delete(minute);
      }
    },
    onCellEnter(day, minute) {
      if (this.isSelecting) {
        this.toggleSelection(day, minute);
      }
    },
    getSelectedIntervals() {
      const intervals = {};
      for (const day in this.selectedMinutes) {
        intervals[day] = Array.from(this.selectedMinutes[day]).map(
          (minute) => ({
            bt: minute,
            et: minute + 60,
          })
        );
      }
      return intervals;
    },
    clearSelection() {
      this.selectedMinutes = {};
    },
    mergeIntervals(existingIntervals, newIntervals) {
      const merged = [...existingIntervals, ...newIntervals];
      merged.sort((a, b) => a.bt - b.bt);

      const result = [];
      if (merged.length === 0) return result;

      let current = merged[0];

      for (let i = 1; i < merged.length; i++) {
        if (current.et >= merged[i].bt) {
          current.et = Math.max(current.et, merged[i].et);
        } else {
          result.push(current);
          current = merged[i];
        }
      }
      result.push(current);
      return result;
    },
    selectAllDay(day) {
      const allDayInterval = { bt: 0, et: 1440 };
      if (this.isAllDaySelected(day)) {
        this.schedule[day] = this.schedule[day].filter(
          (interval) => interval.bt !== 0 || interval.et !== 1440
        );
      } else {
        this.schedule[day].push(allDayInterval);
      }
    },
    isAllDaySelected(day) {
      return this.schedule[day]?.some(
        (interval) => interval.bt === 0 && interval.et === 1440
      );
    },
    saveChanges() {
      const newSchedule = { ...this.schedule };

      for (const day in this.selectedMinutes) {
        const selectedIntervals = Array.from(this.selectedMinutes[day]).map(
          (minute) => ({
            bt: minute,
            et: minute + 60,
          })
        );
        newSchedule[day] = this.mergeIntervals(
          newSchedule[day] || [],
          selectedIntervals
        );
      }

      this.$emit("save-changes", newSchedule);
    },
  },
};
</script>

<style scoped>
.calendar-container {
  margin: 20px;
}

.calendar-table {
  border-collapse: collapse;
  width: 100%;
}

.calendar-table th,
.calendar-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
}

.calendar-table th {
  background-color: #f4f4f4;
}

.allDay {
  background-color: #959595;
}

.busyDay {
  background-color: #e2e2e2;
}

input[type="checkbox"] {
  transform: scale(1.2);
}
</style>
