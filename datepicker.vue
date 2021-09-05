<template>

  <div className="col-12 d-inline-block">
    <div className="input-group">
      <div className="input-group-prepend">
        <div className="input-group-text fa fa-calendar"></div>
      </div>
      <input type="text" :class="['form-control' , {'is-invalid' : invalidDate}]" placeholder="date"
             v-model="date"
             :readonly="! editable" @focusin="show" @focusout="hide">
    </div>
    <div className="text-danger text-sm" v-show="invalidDate" v-text="errorMessage"></div>
  </div>
</template>

<script>
import "@majidh1/jalalidatepicker/dist/jalaliDatepicker.css"
import "@majidh1/jalalidatepicker/dist/jalaliDatepicker"
import "bootstrap/dist/css/bootstrap.min.css"

export default {
  name: "datePicker",
  props: {
    'modelValue': String,
    'defaultDate': {
      type: String,
      default: null
    },
    'errorDate': {
      type: String,
      default: null
    },
    'editable': {
      type: Boolean,
      default: false
    },
    'errorMessage': {
      type: String,
      default: "تاریخ وارد شده صحیح نمیباشد : مثال 1400/02/01"
    },
    'showToday': {
      type: Boolean,
      default: false
    },
    'showEmpty': {
      type: Boolean,
      default: false
    },
    'separator': {
      type: String,
      default: "/"
    },
    'daysNames': {
      type: Array,
      default: ["ش", "ی", "د", "س", "چ", "پ", "ج"]
    },
    'monthsNames': {
      type: Array,
      default: ["فروردین", "اردیبهشت", "خرداد", "تیر", "مرداد", "شهریور", "مهر", "آبان", "آذر", "دی", "بهمن", "اسفند"]
    },
    'minDate': {
      type: String,
      default: null
    },
    'maxDate': {
      type: String,
      default: null
    },
  },
  data() {
    return {
      date: "",
      invalidDate: false,
      uuid: "",
      inPanel: false
    }
  },
  created() {
    this.uuid = "di" + new Date().getTime() + Math.ceil(Math.random() * 200)
    if (!this.validateDate(this.jalali(this.modelValue))) {
      this.date = this.jalali(this.getToday());
    } else {
      this.date = this.jalali(this.modelValue);
    }
    this.init()
  },
  methods: {
    init() {
      let data = {
        changeMonthRotateYear: true,
        autoShow: false,
        showTodayBtn: this.showToday,
        showEmptyBtn: this.showEmpty,
        separatorChar: this.separator,
        days: this.days,
        months: this.months,
      }

      if (this.validateDate(this.jalali(this.minDate))) {
        data.minDate = this.convertToValidOptionDate(this.jalali(this.minDate))
      } else if (this.minDate === "today") {
        data.minDate = "today"
      } else {
        data.minDate = this.convertToValidOptionDate("1200/01/01")
      }

      if (this.validateDate(this.jalali(this.maxDate))) {
        data.maxDate = this.convertToValidOptionDate(this.jalali(this.maxDate))
      } else if (this.maxDate === "today") {
        data.maxDate = "today"
      } else {
        data.maxDate = this.convertToValidOptionDate("2000/01/01")
      }


      jalaliDatepicker.startWatch(data);
    },
    show(e) {
      this.init()
      jalaliDatepicker.show(e.target)
      let panel = document.querySelector("jdp-container");
      panel.addEventListener("mouseenter", evt => {
        this.inPanel = true
      })
      panel.addEventListener("mouseleave", evt => {
        this.inPanel = false
      })
    },
    hide() {
      if (!this.inPanel)
        jalaliDatepicker.hide()
    },
    jalali: function (input) {
      try {
        let moment = require('moment-jalaali');
        let date = moment(input, 'YYYY/MM/DD');
        return date.format(`jYYYY${this.separator}jMM${this.separator}jDD`);
      } catch (e) {
        return input;
      }
    },
    gregorian: function (input) {
      try {
        if (input === undefined || input == null || input.length === 0) return "";
        let moment = require('moment-jalaali');
        let date = moment(input, `jYYYY${this.separator}jM${this.separator}jD`);
        return date.format(`YYYY${this.separator}MM${this.separator}DD`);
      } catch (e) {
        return input;
      }
    },
    validateDate(date) {
      return /^[1-4]\d{3}\/((0[1-6]\/((3[0-1])|([1-2][0-9])|(0[1-9])))|((1[0-2]|(0[7-9]))\/(30|31|([1-2][0-9])|(0[1-9]))))$/.test(date);
    },
    getToday() {
      let today = new Date();
      var dd = today.getDate();
      var mm = today.getMonth() + 1;

      var yyyy = today.getFullYear();
      if (dd < 10) {
        dd = '0' + dd;
      }
      if (mm < 10) {
        mm = '0' + mm;
      }
      return yyyy + '/' + mm + '/' + dd;
    },
    convertToValidOptionDate(date) {
      let ret = {}
      let dspl = date.split(this.separator)
      ret.year = parseInt(dspl[0])
      ret.month = parseInt(dspl[1])
      ret.day = parseInt(dspl[2])
      return ret;
    },
  },
  watch: {
    date() {
      this.$emit("update:modelValue", this.gregorian(this.date))

      this.invalidDate = !this.validateDate(this.gregorian(this.date));

      if (!this.invalidDate) {
        this.$emit("update:modelValue", this.gregorian(this.date))
      } else {
        this.$emit("update:modelValue", this.errorDate)
      }
    },
    minDate() {
      let curr = Date.parse(this.gregorian(this.date))
      let min = Date.parse(this.minDate)
      if (curr < min) {
        this.date = this.jalali(this.minDate)
      }
    },
    maxDate() {
      let curr = Date.parse(this.gregorian(this.date))
      let max = Date.parse(this.maxDate)
      if (curr > max) {
        this.date = this.jalali(this.maxDate)
      }
    }
  }
}
</script>

<style scoped></style>
