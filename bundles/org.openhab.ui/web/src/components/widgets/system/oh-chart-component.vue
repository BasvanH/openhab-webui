<template>
  <div class="oh-chart-container" :style="{ height: activeHeight }">
    <chart
      ref="chart"
      v-if="ready"
      :options="options"
      class="oh-chart"
      @click="handleClick"
      :class="{ 'with-tabbar': context.tab, 'with-toolbar': context.analyzer }"
      :theme="$f7.data.themeOptions.dark === 'dark' ? 'dark' : undefined" autoresize />
    <f7-menu class="padding float-right" v-if="periodVisible">
      <f7-menu-item @click="earlierPeriod()" icon-f7="chevron_left" />
      <f7-menu-item v-if="context.component.config.chartType" :text="fixedPeriodLabel" type="text" @click="pickFixedStartDate">
        <input ref="calendarInput" type="text" style="width: 40px; height: 0; visibility: hidden">
      </f7-menu-item>
      <f7-menu-item v-else dropdown :text="period">
        <f7-menu-dropdown right>
          <f7-menu-dropdown-item v-for="p in ['h', '2h', '4h', '12h', 'D', '2D', '3D', 'W', '2W', 'M', '2M', '4M', '6M', 'Y']"
                                 :key="p" @click="setPeriod(p)" href="#" :text="p" />
        </f7-menu-dropdown>
      </f7-menu-item>
      <f7-menu-item @click="laterPeriod()" icon-f7="chevron_right" />
    </f7-menu>
  </div>
</template>

<style lang="stylus">
.oh-chart-container
  position relative
  .oh-chart
    position absolute
    width 100%
    height 100%
</style>

<script>
import mixin from '../widget-mixin'
import chart from '../chart/chart-mixin'
import { actionsMixin } from '../widget-actions'

import dayjs from 'dayjs'
import LocalizedFormat from 'dayjs/plugin/localizedFormat'
dayjs.extend(LocalizedFormat)

// import echarts from 'echarts/lib/echarts'
import 'echarts/lib/chart/line'
import 'echarts/lib/chart/bar'
import 'echarts/lib/chart/gauge'
import 'echarts/lib/chart/heatmap'
import 'echarts/lib/chart/pie'
import 'echarts/lib/chart/scatter'
import 'echarts/lib/component/title'
import 'echarts/lib/component/legend'
import 'echarts/lib/component/legendScroll'
import 'echarts/lib/component/toolbox'
import 'echarts/lib/component/tooltip'
import 'echarts/lib/component/dataZoom'
import 'echarts/lib/component/markLine'
import 'echarts/lib/component/markPoint'
import 'echarts/lib/component/markArea'
import 'echarts/lib/component/visualMap'
import 'echarts/lib/component/calendar'

import ECharts from 'vue-echarts/components/ECharts'

export default {
  mixins: [mixin, chart, actionsMixin],
  components: {
    'chart': ECharts
  },
  computed: {
    activeHeight () {
      const config = this.config || {}
      return config.height || '300px'
    },
    periodVisible () {
      if (!this.config || this.config.periodVisible === undefined) {
        if (this.context.component.slots && this.context.component.slots.series && Array.isArray(this.context.component.slots.series) && this.context.component.slots.series.length) {
          return this.context.component.slots.series[0].component !== 'oh-data-series'
        }
        return true
      }
      return this.config.periodVisible
    },
    fixedPeriodLabel () {
      const startTime = this.startTime
      if (!this.startTime) return ''
      switch (this.context.component.config.chartType) {
        case 'hour':
          return startTime.format('lll')
        case 'day':
          return startTime.format('ll')
        case 'week':
        case 'isoWeek':
          return startTime.format('ll')
        case 'month':
          return startTime.format('MMM YYYY')
        case 'year':
          return startTime.format('YYYY')
        default:
          return startTime.format('ll')
      }
    }
  },
  data () {
    return {
      ready: false,
      calendarPicker: null
    }
  },
  mounted () {
    this.ready = true
  },
  beforeDestroy () {
    if (this.calendarPicker) this.calendarPicker.destroy()
  },
  methods: {
    handleClick (evt) {
      if (evt.seriesIndex !== undefined) {
        if (this.context.component.slots && this.context.component.slots.series && Array.isArray(this.context.component.slots.series) && this.context.component.slots.series.length) {
          let series = this.context.component.slots.series[evt.seriesIndex]
          this.performAction(evt.event, null, series.config, null)
        }
      }
    },
    pickFixedStartDate (evt) {
      const self = this
      const value = this.startTime.toDate()
      this.calendarPicker = this.$f7.calendar.create({
        inputEl: this.$refs.calendarInput,
        value: [value],
        on: {
          change (calendar, value) {
            if (value.length < 1) return
            if (dayjs(value[0]).isSame(self.startTime)) return
            self.setDate(value[0])
          }
        }
      })
      this.calendarPicker.open()
    }
  }
}
</script>
