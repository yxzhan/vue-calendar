<template>
  <div :class="combineClassNames.root">
    <input :class="combineClassNames.input" type="text"
    :value="value" @keydown.stop.prevent @click="show">
    <transition name="fade">
        <component :class="combineClassNames.panel" :is="currentView"
                   keep-alive
                   transition-mode="out-in"
                   :date="now"
                   :display="display"
                   :active="active"
                   :week-text="weekText"
                   :month-text="monthText"
                   :class-names="combineClassNames"
                   @day="day"
                   @month="month"
                   @year="year"
        ></component>
    </transition>
  </div>
</template>

<style lang="less" scoped>
.calendar {
  display: inline-block;
  width: 200px;
}
.panel {
 display: flex;
 flex-wrap: wrap;
 text-align: center;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .15s ease;
}
.fade-enter, .fade-leave-active {
  opacity: 0;
}
</style>

<script lang="babel">
import moment from 'moment'
import days from './days.vue'
import months from './months.vue'
import years from './years.vue'

export default {
  props: {
    format: {
      type: String,
      default: () => 'YYYY/MM/DD'
    },
    value: {
      type: String,
      required: true
    },
    classNames: {
      type: Object
    },
    weekText: {
      type: Array,
      default: () => ['日','一','二','三','四','五','六']
    },
    monthText: {
      type: Array,
      default: () => ['一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月']
    }
  },
  data() {
    let now = moment(this.value, this.format)
    return {
      date: '',
      now,
      currentView: null,
      dateCache: now.clone(),
      blur: e => !this.$el.contains(e.target) && this.hide()
    }
  },
  computed: {
    combineClassNames() {
        let preset = {
            root: 'calendar',
            input: 'calendar-input',
            panel: 'panel',
            box: 'box',
            bar: 'bar',
            day: 'day',
            month: 'month',
            year: 'year'
        }
        return {...preset, ...this.classNames}
    }
  },
  watch:{
    date(val) {
      this.$emit('input', val)
    },
  },
  mounted() {
    window.addEventListener('click', this.blur)
  },
  beforeDestroy() {
    window.removeEventListener('click', this.blur)
  },
  methods: {
    year(val) {
      this.now = this.now.clone().year(val)
    },
    month(val) {
      this.now = this.now.clone().month(val)
    },
    day(val) {
      this.now = this.now.clone().date(val)
      this.date = this.dump()
      this.hide()
    },
    show() {
      this.currentView = 'days'
    },
    display(viewName) {
      this.currentView = viewName
    },
    hide() {
      this.currentView = null
    },
    active(key, val) {
      let {years: Y, months: M, date: D} = this.dateCache.toObject()
      let {years, months} = this.now.toObject()
      let test = {
        year: val === Y,
        month: years === Y && val === M,
        day: years === Y && months === M && val === D,
      }
      return test[key] ? 'active' : ''
    },
    dump() {
      this.dateCache = this.now.clone()
      return this.dateCache.format(this.format)
    }
  },
  components: {
    days,
    months,
    years
  }
}
</script>