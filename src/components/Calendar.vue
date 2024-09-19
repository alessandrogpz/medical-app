<template>
  <div class="calendar">
    <div class="calendar-header">
      <button @click="prevMonth">‹</button>
      <span>{{ monthNames[currentMonth] }} {{ currentYear }}</span>
      <button @click="nextMonth">›</button>
    </div>

    <!-- "Go to Today" Button -->
    <div class="calendar-actions">
      <button @click="goToToday">Go to Today</button>
    </div>

    <!-- Calendar Grid -->
    <div class="calendar-grid">
      <div v-for="day in weekDays" :key="day" class="calendar-day-header">{{ day }}</div>
      <div v-for="day in blankDays" :key="day" class="calendar-blank"></div>
      <div
          v-for="day in daysInMonth"
          :key="day"
          class="calendar-day"
          :class="{ 'current-day': isCurrentDay(day), 'selected-day': isSelectedDay(day), 'future-day': isFutureDay(day) }"
          @click="selectDay(day)"
      >
        {{ day }}
      </div>
    </div>

    <!-- Data Input Form -->
    <div class="data-input-form" v-if="selectedDay">
      <h3 v-if="!isFutureSelected">Tracking for {{ selectedDay }} {{ monthNames[currentMonth] }} {{ currentYear }}</h3>
      <h3 v-else class="error-message">Cannot input data for future dates!</h3>

      <form @submit.prevent="saveData" v-if="!isFutureSelected">
        <div>
          <label for="weight">Weight:</label>
          <input type="number" v-model="currentDayData.weight" id="weight" />
        </div>
        <div>
          <label for="bloodPressure">Blood Pressure:</label>
          <input type="text" v-model="currentDayData.bloodPressure" id="bloodPressure" />
        </div>
        <button type="submit">Save</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentYear: new Date().getFullYear(),
      currentMonth: new Date().getMonth(),
      currentDate: new Date().getDate(),
      today: new Date(),
      selectedDay: null,
      isFutureSelected: false,
      currentDayData: {
        weight: '',
        bloodPressure: ''
      },
      monthNames: [
        "January", "February", "March", "April", "May", "June",
        "July", "August", "September", "October", "November", "December"
      ],
      weekDays: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
      dailyData: JSON.parse(localStorage.getItem('dailyData')) || {} // Retrieve stored data or initialize empty
    };
  },
  computed: {
    daysInMonth() {
      const days = new Date(this.currentYear, this.currentMonth + 1, 0).getDate();
      return Array.from({ length: days }, (_, i) => i + 1);
    },
    blankDays() {
      const firstDayOfMonth = new Date(this.currentYear, this.currentMonth, 1).getDay();
      return Array.from({ length: firstDayOfMonth }, (_, i) => i);
    },
  },
  methods: {
    prevMonth() {
      if (this.currentMonth === 0) {
        this.currentMonth = 11;
        this.currentYear--;
      } else {
        this.currentMonth--;
      }
      this.selectedDay = null; // Clear the selected day when month changes
    },
    nextMonth() {
      if (this.currentMonth === 11) {
        this.currentMonth = 0;
        this.currentYear++;
      } else {
        this.currentMonth++;
      }
      this.selectedDay = null; // Clear the selected day when month changes
    },
    isCurrentDay(day) {
      return (
          this.currentYear === this.today.getFullYear() &&
          this.currentMonth === this.today.getMonth() &&
          day === this.today.getDate()
      );
    },
    isSelectedDay(day) {
      return this.selectedDay === day;
    },
    isFutureDay(day) {
      const selectedDate = new Date(this.currentYear, this.currentMonth, day);
      return selectedDate > this.today;
    },
    goToToday() {
      this.currentYear = this.today.getFullYear();
      this.currentMonth = this.today.getMonth();
      this.currentDate = this.today.getDate();
      this.selectDay(this.currentDate); // Automatically select today's date
    },
    selectDay(day) {
      const selectedDate = new Date(this.currentYear, this.currentMonth, day);
      if (selectedDate > this.today) {
        // If the selected day is in the future, disable form
        this.isFutureSelected = true;
        this.selectedDay = day;
        this.currentDayData = { weight: '', bloodPressure: '' }; // Clear data
      } else {
        // If the selected day is not in the future, enable form
        this.isFutureSelected = false;
        this.selectedDay = day;
        const dateKey = this.getDateKey(day);
        if (this.dailyData[dateKey]) {
          // Load data for the selected day if it exists
          this.currentDayData = { ...this.dailyData[dateKey] };
        } else {
          // Clear the form if there's no data for this day
          this.currentDayData = { weight: '', bloodPressure: '' };
        }
      }
    },
    saveData() {
      const dateKey = this.getDateKey(this.selectedDay);
      this.dailyData[dateKey] = { ...this.currentDayData };
      localStorage.setItem('dailyData', JSON.stringify(this.dailyData));
      alert("Data saved successfully!");
    },
    getDateKey(day) {
      // Creates a unique string key based on the selected day, month, and year
      return `${this.currentYear}-${this.currentMonth + 1}-${day}`;
    }
  }
};
</script>

<style scoped>
.calendar {
  max-width: 400px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}
.calendar-header {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ddd;
}
.calendar-actions {
  text-align: center;
  margin: 10px 0;
}
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10px;
  padding: 10px;
}
.calendar-day-header,
.calendar-day,
.calendar-blank {
  text-align: center;
  padding: 10px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
}
.current-day {
  background-color: #007bff;
  color: white;
}
.selected-day {
  border: 2px solid #007bff;
}
.future-day {
  background-color: #e0e0e0;
  pointer-events: none; /* Disable click events on future days */
}
.data-input-form {
  margin-top: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  background-color: #f9f9f9;
}
.data-input-form form div {
  margin-bottom: 10px;
}
.data-input-form form label {
  margin-right: 10px;
}
.error-message {
  color: red;
}
</style>
