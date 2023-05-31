<template>
  <div class="wrapper">
    <div class="calendar">
      <h1 class="calendar__title">Set Schedule</h1>
      <div class="calendar__table" @mouseleave="stopDrag">
        <div class="calendar__table-header">
          <div class=""></div>
          <div class="">ALL DAY</div>
          <div>00:00</div>
          <div>03:00</div>
          <div>06:00</div>
          <div>09:00</div>
          <div>12:00</div>
          <div>15:00</div>
          <div>18:00</div>
          <div>21:00</div>
        </div>
        <div
          class="calendar__day"
          v-for="(day, dayIndex) in days"
          :key="day"
          @click="checkAndBook"
          @mousedown="startDrag"
          @mousemove="drag"
          @mouseup="stopDrag"
        >
          <div
            class="calendar__day-title"
            :class="{ selected: isAnyHourBooked(dayIndex) }"
          >
            {{ day }}
          </div>
          <div
            class="calendar__book-day"
            @click="bookAllDay"
            :data-all-day="dayIndex"
            :class="{ selected: isAllDayBooked(dayIndex) }"
          ></div>
          <div
            class="calendar__table-item"
            v-for="hour in 24"
            :key="day + hour"
            :data-day="dayIndex"
            :data-hour="hour - 1"
            :data-booked="calendar[dayIndex][hour - 1]"
          ></div>
        </div>
      </div>
      <div class="calendar__actions">
        <button class="clear" @click="clear">Clear</button>
        <button class="save" @click="uploadCalendar">Save Changes</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TheCalendar",
  data() {
    return {
      days: ["mo", "tu", "we", "th", "fr", "sa", "su"],
      calendar: [],
      draging: false,
      currentDragItem: null,
      fetchedCalendar: {},
    };
  },
  methods: {
    checkAndBook(event) {
      let item = event.target;
      if (item.classList.contains("calendar__table-item")) {
        this.book(item);
      }
    },
    book(item) {
      let isItemBooked = this.calendar[item.dataset.day][item.dataset.hour];
      this.calendar[item.dataset.day][item.dataset.hour] = !isItemBooked;
    },
    startDrag(event) {
      let item = event.target;
      if (item.classList.contains("calendar__table-item")) {
        let isItemBooked = this.calendar[item.dataset.day][item.dataset.hour];
        if (!isItemBooked) {
          this.currentDragItem = item;
          this.draging = true;
        }
      }
    },
    stopDrag() {
      this.draging = false;
    },
    drag(event) {
      let item = event.target;
      if (!item.classList.contains("calendar__table-item")) {
        this.draging = false;
      }
      if (this.draging && item != this.currentDragItem) {
        this.calendar[this.currentDragItem.dataset.day][
          this.currentDragItem.dataset.hour
        ] = true;
        this.currentDragItem = item;
        this.calendar[this.currentDragItem.dataset.day][
          this.currentDragItem.dataset.hour
        ] = true;
      }
    },
    bookAllDay(event) {
      let day = event.target.dataset.allDay;
      if (!this.isAllDayBooked(day)) {
        for (let hour = 0; hour < this.calendar[day].length; hour++) {
          this.calendar[day][hour] = true;
        }
      } else {
        for (let hour = 0; hour < this.calendar[day].length; hour++) {
          this.calendar[day][hour] = false;
        }
      }
    },
    clear() {
      this.createEmptyTable();
    },
    createEmptyTable() {
      for (let index = 0; index < this.days.length; index++) {
        this.calendar[index] = [];
        for (let hour = 0; hour < 24; hour++) {
          this.calendar[index][hour] = false;
        }
      }
    },
    fetchCalendar() {
      if (localStorage.calendar) {
        this.fetchedCalendar = JSON.parse(localStorage.calendar);
        this.implementFetchedCalendar();
      } else {
        fetch("https://kotyhoroshko.github.io/calendar.json")
          .then((response) => {
            return response.json();
          })
          .then((data) => {
            this.fetchedCalendar = data;
            this.implementFetchedCalendar();
          });
      }
    },
    implementFetchedCalendar() {
      for (let day = 0; day < this.days.length; day++) {
        let dayIndex = this.fetchedCalendar[this.days[day]];
        if (dayIndex.length > 0) {
          for (let book = 0; book < dayIndex.length; book++) {
            for (
              let hour = dayIndex[book].bt / 60;
              hour < (dayIndex[book].et + 1) / 60;
              hour++
            ) {
              this.calendar[day][hour] = true;
            }
          }
        }
      }
    },
    uploadCalendar() {
      let updatedCalendar = {};
      let bt = false;
      for (let day = 0; day < this.calendar.length; day++) {
        updatedCalendar[this.days[day]] = [];
        for (let hour = 0; hour < this.calendar[day].length; hour++) {
          if (this.calendar[day][hour] && bt === false) {
            bt = hour * 60;
          } else if (!this.calendar[day][hour] && bt !== false) {
            updatedCalendar[this.days[day]].push({ bt: bt, et: hour * 60 - 1 });
            bt = false;
          } else if (
            this.calendar[day][hour + 1] == undefined &&
            bt !== false
          ) {
            updatedCalendar[this.days[day]].push({
              bt: bt,
              et: (hour + 1) * 60 - 1,
            });
            bt = false;
          }
        }
        localStorage.setItem("calendar", JSON.stringify(updatedCalendar));
      }
    },
    isAllDayBooked(dayIndex) {
      if (this.calendar[dayIndex].find((hour) => hour == false) == false) {
        return false;
      } else {
        return true;
      }
    },
    isAnyHourBooked(dayIndex) {
      if (this.calendar[dayIndex].find((hour) => hour == true) == true) {
        return true;
      }
      return false;
    },
  },
  beforeMount() {
    this.createEmptyTable();
    this.fetchCalendar();
  },
};
</script>

<style lang="scss">
.wrapper {
  padding: 4vmin 2vmin;
}

.calendar {
  &__title {
    margin-bottom: 1rem;
    text-transform: uppercase;
    text-align-last: left;
    font-weight: 100;
  }

  &__table {
    display: flex;
    flex-wrap: wrap;
    border-bottom: 1px solid gray;
  }

  &__table-header {
    display: flex;
    width: 100%;
    user-select: none;

    div {
      position: relative;
      display: flex;
      width: calc((100% - 80px) / 8);
      align-items: center;
      justify-content: flex-start;
      padding: 4px;

      &:before {
        content: "";
        display: block;
        position: absolute;
        height: 75%;
        width: 2px;
        bottom: 0;
        left: 0;
        background: #949494;
      }

      &:nth-child(1),
      &:nth-child(2) {
        width: 40px;
        padding: 4px;

        &:before {
          content: none;
        }
      }
    }
  }

  &__day {
    display: flex;
    width: 100%;
    border-left: 1px solid gray;
    border-right: 1px solid gray;

    &:nth-of-type(2) .calendar__book-day {
      border-top: 1px solid #949494
    }
  }

  &__table-item {
    border-top: 1px solid #dcdcdc;
    border-left: 1px solid gray;
    width: calc((100% - 80px) / 24);
    user-select: none;
    cursor: pointer;

    &[data-booked="true"],
    &[data-reserved="true"] {
      background: #dbdbdb;
    }
  }

  &__day-title,
  &__book-day {
    min-width: 40px;
    padding: 4px;
    border-top: 1px solid #949494;
    border-right: 1px solid gray;
    text-transform: uppercase;
    user-select: none;
  }
  
  &__day-title.selected {
    background: #dcdcdc;
  }
  
  &__book-day {
    background-color: #949494;
    border-top: 1px solid white;
    cursor: pointer;
    transition: 0.2s ease;
    
    &.selected {
      background: #949494 url(../assets/done.svg) no-repeat center;
      background-size: 1.25rem;
    }
  }
  
  &__actions {
    margin-top: 2rem;

    .clear,
    .save {
      background: #949494;
      color: white;
      border: none;
      border-radius: 0.25rem;
      margin-inline: 0.25rem;
      padding: 0.25rem 1rem;
      cursor: pointer;
    }
  }
}
</style>
