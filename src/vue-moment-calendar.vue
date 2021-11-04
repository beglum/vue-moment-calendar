<template>
  <div class="moment-calendar">
    <div class="navigation">
      <button class="button" @click="toPreviousMonth">&lsaquo;</button>
      <span class="current-month-name bold">{{ titleCurrentMonth }}</span>
      <button class="button" @click="toNextMonth">&rsaquo;</button>
    </div>
    <div class="week-names bold">
      <div
        v-for="(weekName, index) of uppercaseWeekDays"
        :key="index"
        class="week-name"
        :class="{
          weekend: index >= 5,
        }"
      >
        {{ weekName }}
      </div>
    </div>
    <div class="calendar-days">
      <div
        v-for="(day, index) of displayedDaysOfCurrentCalendarPage"
        :key="index"
        class="calendar-day"
        :class="{
          weekend: day.isWeekend,
          'current-day': day.isToday,
          'previous-day': day.isPreviousDay,
        }"
      >
        <div class="day-number bold">{{ day.date.date() }}</div>
        <div class="event-list">
          <div
            class="event-container"
            v-for="event of day.events"
            :key="event.id"
          >
            <div class="day-event" :class="event.type">
              {{ event.label }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment';

export default /*#__PURE__*/ {
  name: 'VueMomentCalendar', // vue component name
  data: () => ({
    chosenMonth: null,
    initialDate: null,
  }),
  props: {
    events: Array,
    lang: {
      type: String,
      default: 'ru',
    },
  },
  computed: {
    // Список названий дней в неделе в верхнем регистре
    uppercaseWeekDays() {
      return moment.weekdays(true).map((name) => name.toUpperCase());
    },
    // Порядковый номер текущего месяца
    currentMonthNumber() {
      return this.chosenMonth.month();
    },
    isCurrentYear() {
      return this.chosenMonth.isSame(this.initialDate, 'year');
    },
    currentMonth() {
      return this.chosenMonth;
    },
    previousMonth() {
      return this.chosenMonth.clone().subtract(1, 'M');
    },
    nextMonth() {
      return this.chosenMonth.clone().add(1, 'M');
    },
    titleCurrentMonth() {
      let title = this.currentMonth.format('MMMM').toUpperCase();
      if (!this.isCurrentYear) {
        title += ` ${this.currentMonth.year()}`;
      }

      return title;
    },
    weekDayOfStartsCurrentMonth() {
      let dateObject = this.chosenMonth.clone().date(1);
      return +dateObject.format('E');
    },
    weekDayOfEndsCurrentMonth() {
      let dateObject = this.chosenMonth
        .clone()
        .date(this.chosenMonth.daysInMonth());
      return +dateObject.format('E');
    },
    // Объект, хранящий список событий текущего месяца, с ключами
    eventsByKeyInCurrentMonth() {
      let eventsListByKey = {};
      this.events.forEach((event) => {
        const dateObject = this.getMomentObjectByDateString(event.date);
        if (dateObject.year() === this.currentMonth.year()) {
          const hours = dateObject.hour().toString().padStart(2, '0');
          const minutes = dateObject.minute().toString().padStart(2, '0');
          const data = {
            id: event.id,
            label: `${hours}:${minutes} ${event.name}`,
            type: event.type,
          };

          const key = this.generateDateStringByAttrsList([
            dateObject.date(),
            dateObject.month() + 1,
            dateObject.year(),
          ]);
          const events = eventsListByKey[key] || [];
          events.push(data);
          eventsListByKey[key] = events;
        }
      });

      return eventsListByKey;
    },

    displayedDaysOfCurrentCalendarPage() {
      let currentMonth = this.currentMonth;
      let daysFromPreviousMonth = this.weekDayOfStartsCurrentMonth - 1;
      let daysFromNextMonth = 7 - this.weekDayOfEndsCurrentMonth;

      let events = [];

      // Сначала заполняем недостающие дни из прошлым месяцем
      for (let i = daysFromPreviousMonth - 1; i >= 0; i--) {
        let dateString = this.generateDateStringByAttrsList([
          this.previousMonth.daysInMonth() - i,
          this.previousMonth.month() + 1,
          this.previousMonth.year(),
        ]);
        let data = this.generateDayObject(dateString);

        events.push(data);
      }

      // Добавляем дни текущего месяца
      for (let dayNum = 1; dayNum <= currentMonth.daysInMonth(); dayNum++) {
        let dateString = this.generateDateStringByAttrsList([
          dayNum,
          this.currentMonthNumber + 1,
          this.chosenMonth.year(),
        ]);
        let data = this.generateDayObject(dateString);

        events.push(data);
      }

      // Заполняем недостающие дни из следующего месяца
      for (let dayNum = 1; dayNum <= daysFromNextMonth; dayNum++) {
        let dateString = this.generateDateStringByAttrsList([
          dayNum,
          this.nextMonth.month() + 1,
          this.nextMonth.year(),
        ]);
        let data = this.generateDayObject(dateString);

        events.push(data);
      }

      return events;
    },
  },
  methods: {
    getEventsByKey(key) {
      return this.eventsByKeyInCurrentMonth[key] || [];
    },
    getMomentObjectByDateString(dateString) {
      return moment(dateString, 'DD-MM-YYYY hh:mm:ss');
    },
    generateDateStringByAttrsList(attrs) {
      return attrs.join('-');
    },
    generateDayObject(dateString) {
      const dateObject = this.getMomentObjectByDateString(dateString);
      return {
        date: dateObject,
        events: this.getEventsByKey(dateString),
        weekDay: +dateObject.format('E'),
        isWeekend: +dateObject.format('E') > 5,
        isToday: this.initialDate.isSame(dateObject, 'day'),
        isPreviousDay: this.initialDate.isAfter(dateObject, 'day'),
      };
    },
    toPreviousMonth() {
      this.chosenMonth = this.previousMonth;
    },
    toNextMonth() {
      this.chosenMonth = this.nextMonth;
    },
  },
  created() {
    moment.locale(this.lang);
    this.chosenMonth = moment().locale(this.lang);
    this.initialDate = moment().locale(this.lang);
  },
};
</script>

<style scoped lang="scss">
$gray: rgb(248, 248, 248);
$white: white;
$weekend: #b6bdd9;
$weekend-week-name: #9a9a9a;
$current-day-border: #b9b9b9;
$current-day-number: #91b89b;
$previous-day-bg: #f5f5f5;
$previous-day-border: #f2f2f2;
$previous-day-number: #9b9b9b;
$bage-red-bg: #fdebed;
$bage-red-color: #bd8a8d;
$bage-green-bg: #e5f8e9;
$bage-green-color: #86b792;
$bage-orange-bg: #fcebce;
$bage-orange-color: #bd9f6f;

* {
  box-sizing: border-box;
}

.bold {
  font-weight: 900;
}

.navigation {
  margin-bottom: 12px;

  .button {
    border: none;
    background: none;
    width: 24px;
    cursor: pointer;
    font-size: 1.3em;
  }

  .current-month-name {
    color: #606fa7;
  }
}

.week-name {
  text-align: right;
  font-size: 0.8em;
  &.weekend {
    color: $weekend-week-name;
  }
}

.week-names,
.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, minmax(10px, 1fr));
  gap: 1px;
  padding: 2px;
}

.calendar-day {
  background-color: $white;
  border-radius: 8px;
  min-height: 100px;
  border: 2px solid $gray;
  padding: 6px;

  &.weekend {
    color: $weekend;
  }

  .day-number {
    text-align: right;
    top: -2px;
    position: relative;
  }

  .event-list {
    display: flex;
    flex-direction: column;
    gap: 4px;

    .event-container {
      position: relative;
      height: 1.3em;

      .day-event {
        white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        z-index: 1;
        padding: 2px 4px;
        border-radius: 6px;

        &:hover {
          position: absolute;
          overflow: auto;
        }

        &.red {
          background-color: $bage-red-bg;
          color: $bage-red-color;
        }
        &.green {
          background-color: $bage-green-bg;
          color: $bage-green-color;
        }
        &.orange {
          background-color: $bage-orange-bg;
          color: $bage-orange-color;
        }
      }
    }
  }

  &.previous-day {
    border-color: $previous-day-border;
    background-color: $previous-day-bg;
    .day-number {
      color: $previous-day-number;
    }
  }

  &.current-day {
    border-color: $current-day-border;
    .day-number {
      color: $current-day-number;
    }
  }
}
</style>
