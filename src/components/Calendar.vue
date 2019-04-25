<template>
  <table class="table-condensed">
    <thead>
      <tr>
        <th class="prev available" @click="$emit('prevMonth')">
          <i :class="[arrowLeftClass]"></i>
        </th>
        <th colspan="5" class="month">
          <select class="dp-dropdown" name="months" v-if="this.showDropdowns" @change="changeMonth">
            <option
              v-for="(monthItem, index) in locale.monthNames"
              :value="monthItem"
              :key="index"
              :selected="monthItem === monthName"
            >{{monthItem}}</option>
          </select>
          <span v-else>{{monthName}}</span>
          <select class="dp-dropdown" name="years" v-if="this.showDropdowns" @change="changeYear">
            <option
              v-for="(yearItem, index) in locale.years"
              :value="yearItem"
              :key="index"
              :selected="yearItem === year  "
            >{{yearItem}}</option>
          </select>
          <span v-else>{{year}}</span>
        </th>
        <th class="next available" @click="$emit('nextMonth')">
          <i :class="[arrowRightClass]"></i>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th v-for="weekDay in locale.daysOfWeek" :key="weekDay">{{weekDay}}</th>
      </tr>
      <tr v-for="(dateRow, index) in calendar" :key="index">
        <slot name="date-slot" v-for="(date, idx) in dateRow">
          <td
            :class="dayClass(date)"
            @click="$emit('dateClick', date)"
            @mouseover="$emit('hoverDate', date)"
            :key="idx"
          >{{date | dateNum}}</td>
        </slot>
      </tr>
    </tbody>
  </table>
</template>

<script>
import moment from "moment";

export default {
  name: "calendar",
  props: [
    "date",
    "locale",
    "isStartDate",
    "start",
    "end",
    "minDate",
    "maxDate",
    "showDropdowns"
  ],
  // data() {
  //   return {
  //     month:
  //   }
  // },
  methods: {
    changeMonth(e) {
      this.$emit("changeMonth", e.target.value, this.isStartDate);
    },
    changeYear(e) {
      this.$emit("changeYear", e.target.value, this.isStartDate);
    },
    dayClass(date) {
      let dt = new Date(date);
      dt.setHours(0, 0, 0, 0);
      let start = new Date(this.start);
      start.setHours(0, 0, 0, 0);
      let end = new Date(this.end);
      end.setHours(0, 0, 0, 0);

      return {
        off: date.month() !== this.month,
        weekend: date.isoWeekday() > 5,
        today: dt.setHours(0, 0, 0, 0) == new Date().setHours(0, 0, 0, 0),
        active:
          dt.setHours(0, 0, 0, 0) ==
            new Date(this.start).setHours(0, 0, 0, 0) ||
          dt.setHours(0, 0, 0, 0) == new Date(this.end).setHours(0, 0, 0, 0),
        "in-range": dt >= start && dt <= end,
        "start-date": dt.getTime() === start.getTime(),
        "end-date": dt.getTime() === end.getTime(),
        disabled:
          (this.minDate &&
            moment(dt)
              .startOf("day")
              .isBefore(moment(this.minDate).startOf("day"))) ||
          (this.maxDate &&
            moment(dt)
              .startOf("day")
              .isAfter(moment(this.maxDate).startOf("day")))
      };
    }
  },
  computed: {
    arrowLeftClass() {
      return "ocpx-icon-navback";
    },
    arrowRightClass() {
      return "ocpx-icon-navforward";
    },
    monthName() {
      return this.locale.monthNames[this.date.getMonth()];
    },
    year() {
      return this.date.getFullYear();
    },
    month() {
      return this.date.getMonth();
    },
    calendar() {
      let month = this.month;
      let year = this.date.getFullYear();
      let daysInMonth = new Date(year, month, 0).getDate();
      let firstDay = new Date(year, month, 1);
      let lastDay = new Date(year, month, daysInMonth);
      let lastMonth = moment(firstDay)
        .subtract(1, "month")
        .month();
      let lastYear = moment(firstDay)
        .subtract(1, "month")
        .year();
      let daysInLastMonth = moment([lastYear, lastMonth]).daysInMonth();

      let dayOfWeek = firstDay.getDay();

      let calendar = [];

      for (let i = 0; i < 6; i++) {
        calendar[i] = [];
      }

      let startDay = daysInLastMonth - dayOfWeek + this.locale.firstDay + 1;
      if (startDay > daysInLastMonth) startDay -= 7;

      if (dayOfWeek === this.locale.firstDay) startDay = daysInLastMonth - 6;

      let curDate = moment([lastYear, lastMonth, startDay, 12, 0, 0]);
      for (
        let i = 0, col = 0, row = 0;
        i < 6 * 7;
        i++, col++, curDate = moment(curDate).add(1, "day")
      ) {
        if (i > 0 && col % 7 === 0) {
          col = 0;
          row++;
        }
        calendar[row][col] = curDate.clone();
        curDate.hour(12);
      }

      return calendar;
    }
  },
  filters: {
    dateNum(value) {
      return value.date();
    }
  },
  mounted() {
    if (this.showDropdowns && this.locale.years.length === 0) {
      let years = moment(this.maxDate).diff(this.minDate, "years");
      if (years === 0) {
        this.locale.years.push(moment().format("YYYY"));
      }
      for (let year = 0; year < years; year++) {
        this.locale.years.push(this.minDate.getFullYear() + year);
      }
    }
  }
};
</script>

<style scoped lang="scss">
.dp-dropdown {
  margin-right: 18px;
}
td.today {
  font-weight: bold;
}

td.disabled {
  pointer-events: none;
  background-color: #eee;
  border-radius: 0;
  opacity: 0.6;
}

@function str-replace($string, $search, $replace: "") {
  $index: str-index($string, $search);

  @if $index {
    @return str-slice($string, 1, $index - 1) + $replace +
      str-replace(
        str-slice($string, $index + str-length($search)),
        $search,
        $replace
      );
  }

  @return $string;
}

$carousel-control-color: #ccc !default;
$viewbox: "-2 -2 10 10";
$carousel-control-prev-icon-bg: str-replace(
  url("data:image/svg+xml;charset=utf8,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='#{$viewbox}'%3E%3Cpath d='M5.25 0l-4 4 4 4 1.5-1.5-2.5-2.5 2.5-2.5-1.5-1.5z'/%3E%3C/svg%3E"),
  "#",
  "%23"
) !default;
$carousel-control-next-icon-bg: str-replace(
  url("data:image/svg+xml;charset=utf8,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='#{$viewbox}'%3E%3Cpath d='M2.75 0l-1.5 1.5 2.5 2.5-2.5 2.5 1.5 1.5 4-4-4-4z'/%3E%3C/svg%3E"),
  "#",
  "%23"
) !default;

.fa {
  display: inline-block;
  width: 100%;
  height: 100%;
  background: transparent no-repeat center center;
  background-size: 100% 100%;
  fill: $carousel-control-color;
}

.prev,
.next {
  &:hover {
    background-color: transparent !important;
  }

  .fa:hover {
    opacity: 0.6;
  }
}

.chevron-left {
  width: 16px;
  height: 16px;
  display: block;
  background-image: $carousel-control-prev-icon-bg;
}
.chevron-right {
  width: 16px;
  height: 16px;
  display: block;
  background-image: $carousel-control-next-icon-bg;
}
</style>
