<template>
  <div class="demo container">
    <div class="text-center">
      <img src="./assets/logo.png">

      <h1>Vue Date Range Picker</h1>
      <p>
        Based on
        <a
          href="http://www.daterangepicker.com"
          target="_blank"
        >http://www.daterangepicker.com</a>
      </p>

      <p>
        The component is rewritten without jQuery dependancy. Requires only bootstrap and the original
        datepicker CSS (included).
      </p>
    </div>

    <div class="card bg-light" style="margin-bottom: 300px;">
      <div class="card-body">
        <div class="row">
          <div class="col-md-6">
            <div class="form-group row">
              <label class="col-sm-4 col-form-label" for="startDate">StartDate</label>
              <div class="col-sm-8">
                <input type="text" class="form-control" id="startDate" v-model="startDate">
              </div>
            </div>
            <div class="form-group row">
              <label class="col-sm-4 col-form-label" for="endDate">EndDate</label>
              <div class="col-sm-8">
                <input type="text" class="form-control" id="endDate" v-model="endDate">
              </div>
            </div>
          </div>
          <div class="col-md-6">
            <div class="form-group row">
              <label class="col-sm-4 col-form-label" for="minDate">minDate</label>
              <div class="col-sm-8">
                <input type="text" class="form-control" id="minDate" v-model="minDate">
              </div>
            </div>
            <div class="form-group row">
              <label class="col-sm-4 col-form-label" for="maxDate">maxDate</label>
              <div class="col-sm-8">
                <input type="text" class="form-control" id="maxDate" v-model="maxDate">
              </div>
            </div>
          </div>
        </div>

        <div class="form-group">
          <label class="col-form-label">Opens:</label>
          <div class="form-check form-check-inline">
            <input
              class="form-check-input"
              type="radio"
              name="options"
              id="option1"
              value="left"
              v-model="opens"
            >
            <label class="form-check-label">left</label>
          </div>
          <div class="form-check form-check-inline">
            <input
              class="form-check-input"
              type="radio"
              name="options"
              id="option2"
              value="center"
              v-model="opens"
            >
            <label class="form-check-label">center</label>
          </div>
          <div class="form-check form-check-inline">
            <input
              class="form-check-input"
              type="radio"
              name="options"
              id="option3"
              value="right"
              v-model="opens"
            >
            <label class="form-check-label">right</label>
          </div>
        </div>
        <span
          class="form-text text-muted"
        >(string: 'left'/'right'/'center') Whether the picker appears aligned to the left, to the right, or centered under the HTML element it's attached to</span>

        <div class="py-5">
          <h1>Demo</h1>
          <div class="form-group">
            <label>Select range:</label>
            <date-range-picker
              footerInputIconClassName="ocpx ocpx-icon-agenda"
              :class="{singlePicker: singleDatePicker}"
              :headlineTitle="headlineTitle"
              :opens="opens"
              :startDate="startDate"
              :endDate="endDate"
              :locale-data="{ firstDay: 1, format: 'DD.MM.YYYY HH:mm' }"
              :hours="true"
              :footerInputs="true"
              :headline="true"
              :minDate="minDate"
              :maxDate="maxDate"
              :singleDatePicker="singleDatePicker"
              @updateDatePicker="updateDatePicker"
            >
              <!-- <div slot="input" slot-scope="picker">
                <span>{{ picker.startDate | date }}</span>
                <span v-if="!singleDatePicker">{{' - ' + picker.endDate | date}}</span>
              </div>-->
            </date-range-picker>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import DateRangePicker from "./components/DateRangePicker";
import moment from "moment";

export default {
  components: { DateRangePicker },
  name: "DateRangePickerDemo",
  filters: {
    date(value) {
      return moment(value).format("DD.MM.YYYY HH:mm");
      // let options = {year: 'numeric', month: 'long', day: 'numeric'};
      // return Intl.DateTimeFormat('en-US', options).format(value)
    }
  },
  data() {
    //                    :locale-data="{ daysOfWeek: [ 'Нд', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб' ] }"
    return {
      opens: "center",
      startDate: moment().format("MM.DD.YYYY HH:mm"),
      endDate: moment()
        .add(1, "month")
        .format("MM.DD.YYYY HH:mm"),
      minDate: moment()
        .subtract(10, "years")
        .format("MM.DD.YYYY HH:mm"),
      maxDate: moment()
        .add(10, "years")
        .format("MM.DD.YYYY HH:mm"),
      singleDatePicker: false,
      headlineTitle: "Validity start / end"
    };
  },
  mounted() {
    if (this.singleDatePicker) {
      this.headlineTitle = "Validity date";
    }
  },
  methods: {
    updateDatePicker(dates) {
      const { startDate, endDate } = dates;
      this.startDate = moment(startDate.toISOString()).format(
        "MM.DD.YYYY HH:mm"
      );
      this.endDate = moment(endDate.toISOString()).format("MM.DD.YYYY HH:mm");
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  /*margin: 0 10px;*/
  width: 100%;
}

a {
  color: #42b983;
}
</style>
