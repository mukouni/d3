<template>
  <div>
    <svg @mousemove="mouseover" :width="width" :height="height" id="d3-svg">
      <g id="x-axis" :style="{ transform: `translate(${0}px, ${padded.height + 10}px)` }" />
      <g
        id="y-axis"
        :style="{ transform: `translate(${margin.left}px, ${margin.top}px)` }"
        ref="yAxis"
      />
      <g :style="{ transform: `translate(${paddingOuter}px, ${margin.top}px)` }">
        <path class="area" :d="paths.area" />
        <template v-for="(line, index) in paths.lines">
          <path
            class="line"
            :d="line"
            :stroke="animatedData[index][0].color"
            :key="index"
            fill="none"
            stroke-width="1.5"
            @click="(index)=>clickLine(index)"
          />
        </template>
        <path class="selector" :d="paths.selector" />
      </g>
      <!-- <template v-for="(line, index) in animatedData">
        <g :style="{fill: line[0].color}" :key="index" :id="`points${index}`"></g>
      </template>-->
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'

import TWEEN from 'tween.js'
import moment from 'moment'
export default {
  name: 'timeLineChart',
  props: {
    data: {
      type: Array,
      default: () => [100, 2, 13, 41, 5, 16, 71, 18, 19, 10]
    },
    margin: {
      type: Object,
      default: () => ({
        left: 30,
        right: 30,
        top: 10,
        bottom: 30
      })
    },
    ceil: {
      type: Number,
      default: 100
    }
  },
  data () {
    return {
      svg: null,
      yAxisWidth: 30,
      viewDates: [],
      width: 0,
      height: 200,
      paths: {
        area: '',
        lines: [],
        selector: ''
      },
      lastHoverPoint: {},
      scaled: {
        x: null,
        y: null
      },
      animatedData: [],
      points: [],
      paddingOuter: 0
    }
  },
  computed: {
    padded () {
      const width = this.width - this.margin.left - this.margin.right
      const height = this.height - this.margin.top - this.margin.bottom
      return { width, height }
    }
  },

  watch: {
    data: function dataChanged (newData, oldData) {},
    width: function widthChanged (data) {
      this.updateAxis()
      this.updateLine()
    },
    animatedData (val) {
      this.viewDates = this.animatedData
        .reduce((all, line) => {
          let lineDates = line.map(point => moment(point.date).valueOf())
          all = Array.from(new Set(all.concat(lineDates)))
          return all
        }, [])
        .sort((a, b) => {
          return a - b
        })

      this.updateAxis()
      this.updateLine()
    }
  },
  methods: {
    onResize () {
      this.width = this.$el.offsetWidth
      // this.height = this.$el.offsetHeight;
    },
    tweenData (newData, oldData) {
      // const vm = this
      function animate (time) {
        requestAnimationFrame(animate)
        TWEEN.update(time)
      }
      // new TWEEN.Tween(oldData)
      //   .easing(TWEEN.Easing.Quadratic.Out)
      //   .to(newData, 500)
      //   .onUpdate(function onUpdate() {
      //     vm.animatedData = this;
      //     vm.update();
      //   })
      //   .start();
      animate()
    },

    updateAxis () {
      if (this.padded.width < 0) return
      let labelWidth = 60 // x label Width 60px
      let allDate = d3.timeDay
        .range(
          this.viewDates[0],
          moment(this.viewDates.slice(-1)[0]).add(1, 'day')
        )
        .map(date => {
          return moment(date).toDate()
        })
      let maxAxisLabelNum = Math.floor(this.padded.width / labelWidth + 0) - 1
      let firstIndex = 0
      let scaleWidth = this.padded.width / (allDate.length + 1)
      if (scaleWidth < labelWidth / 2) {
        firstIndex = Math.ceil(labelWidth / 2 / scaleWidth) - 1
      }
      let stepWidth = Math.ceil(allDate.length / maxAxisLabelNum)
      let xAxisLabels = []
      if (maxAxisLabelNum < allDate.length) {
        let beforeIndex = firstIndex
        allDate.forEach((date, index) => {
          if (index === firstIndex || index === beforeIndex + stepWidth) {
            beforeIndex = index
            xAxisLabels.push(date)
          }
        })
      } else {
        xAxisLabels = allDate
      }
      let scaleBand = d3
        .scaleBand()
        .range([this.margin.left, this.padded.width + this.margin.left])
        .domain(allDate)
        .paddingOuter(0.5) // scaleWidth * 0.5 px
      this.paddingOuter = scaleWidth * 0.5

      this.scaled.x = scaleBand
      // this.scaled.x = d3
      //   .scaleLinear()
      //   .range([this.margin.left, this.padded.width + this.margin.left])
      //   .domain([this.viewDates[0], this.viewDates.slice(-1)])

      this.scaled.y = d3
        .scaleLinear()
        .range([this.padded.height, 0])
        .domain([
          this.minValue(this.animatedData),
          this.maxValue(this.animatedData)
        ])
        .nice()

      this.svg
        .selectAll('#x-axis')
        // .attr('transform', 'translate(30,' + (this.padded.height + 10) + ')')
        .call(
          d3
            .axisBottom(scaleBand)
            .tickFormat((date, i) => {
              if (
                i === 0 ||
                new Date(xAxisLabels[i - 1]).getFullYear() <
                  new Date(date).getFullYear()
              ) {
                return d3.timeFormat('%Y/%m/%d')(date)
              } else {
                return d3.timeFormat('%m/%d')(date)
              }
            })
            .tickValues(xAxisLabels)
        )
      this.svg
        .selectAll('#y-axis')
        // .attr('transform', 'translate(' + 30 + ',10)')
        .call(d3.axisLeft(this.scaled.y).ticks(3))
    },
    updateLine () {
      this.points = []
      for (const [lineIndex, data] of this.animatedData.entries()) {
        if (!this.points[lineIndex]) {
          this.points[lineIndex] = []
        }
        data.forEach((data, index) => {
          let findIndex = this.viewDates.findIndex(
            date => date === moment(data.date).valueOf()
          )
          if (findIndex > -1) {
            this.points[lineIndex].push(
              Object.assign(data, {
                x: this.scaled.x(moment(data.date).toDate()),
                y: this.scaled.y(data.value),
                max: this.height
              })
            )
          }
        })
      }

      // this.paths.area = this.createArea(this.points)
      this.points.forEach((line, index) => {
        if (!this.paths.lines[index]) {
          this.paths.lines[index] = {}
        }
        this.paths.lines[index] = this.createLine(line)
      })
    },
    clickLine (index) {
    },
    mouseover ({ offsetX }) {
      if (this.points.length > 0) {
        const x = offsetX - this.margin.left
        const closestPoint = this.getClosestPoint(x)
        if (this.lastHoverPoint.index !== closestPoint.index) {
          // const point = this.points[closestPoint.index]
          // this.paths.selector = this.createValueSelector([point]);
          this.$emit('select', this.data[closestPoint.index])
          this.lastHoverPoint = closestPoint
        }
      }
    },
    createArea: d3
      .area()
      .x(d => d.x)
      .y0(d => d.max)
      .y1(d => d.y),

    createLine: d3
      .line()
      .x(d => d.x)
      .y(d => d.y),

    createValueSelector: d3
      .area()
      .x(d => d.x)
      .y0(d => d.max)
      .y1(0),
    maxDate (lines) {
      let maxDate = ''
      lines.forEach((line, index) => {
        let lineMax = d3.max(line, d => moment(d.date).valueOf())
        if (lineMax > maxDate || index === 0) {
          maxDate = lineMax
        }
      })
      return maxDate
    },
    minDate (lines) {
      let minDate = ''
      lines.forEach((line, index) => {
        let lineMin = d3.min(line, d => moment(d.date).valueOf())
        if (lineMin < minDate || index === 0) {
          minDate = lineMin
        }
      })
      return minDate
    },
    maxValue (lines) {
      let maxValue = ''
      lines.forEach((line, index) => {
        let lineMax = d3.max(line, d => d.value)
        if (lineMax > maxValue || index === 0) {
          maxValue = lineMax
        }
      })
      return maxValue
    },
    minValue (lines) {
      let minValue = ''
      lines.forEach((line, index) => {
        let lineMin = d3.min(line, d => d.value)
        if (lineMin < minValue || index === 0) {
          minValue = lineMin
        }
      })
      return minValue
    },
    getClosestPoint (x) {
      return this.points
        .map((point, index) => ({
          x: point.x,
          diff: Math.abs(point.x - x),
          index
        }))
        .reduce((memo, val) => (memo.diff < val.diff ? memo : val))
    }
  },

  mounted () {
    this.svg = d3.select('#d3-svg')

    this.animatedData = []
    let color = ['#7cb55d', '#5d72b5', '#d9ca64', '#b5785d', '#5daeb5'];
    [...new Array(5)].forEach((_, lineIndex) => {
      if (!this.animatedData[lineIndex]) this.animatedData[lineIndex] = [];
      [...new Array(3 * 7)].forEach((_, index) => {
        if (lineIndex === 0 || lineIndex === 4) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex]
          }
        }
        if (lineIndex === 1 && index % 7 < 5) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex]
          }
        }

        if (lineIndex === 2 && index % 7 < 2) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex]
          }
        }

        if (lineIndex === 3 && index % 7 < 1) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex]
          }
        }
      })
    })

    this.updateAxis()
    window.addEventListener('resize', this.onResize)
    this.onResize()
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
