<template>
  <svg width="500" height="270">
    <g style="transform: translate(0, 10px)">
      <path :d="line" />
      <path :d="line1" />
    </g>
  </svg>
</template>

<script>
import * as d3 from 'd3'
export default {
  name: 'vue-line-chart',
  data () {
    return {
      data: [99, 71, 78, 25, 36, 92],
      data1: [99, 51, 8, 21, 26, 42],
      line: '',
      line1: ''
    }
  },
  mounted () {
    this.calculatePath()
  },
  methods: {
    getScales (data) {
      console.log(d3.extent(data, (d, i) => i))
      const x = d3.scaleTime().range([0, 430])
      const y = d3.scaleLinear().range([210, 0])
      d3.axisLeft().scale(x)
      d3.axisBottom().scale(y)
      x.domain(d3.extent(data, (d, i) => i))
      y.domain([0, d3.max(data, d => d)])
      return { x, y }
    },
    calculatePath () {
      const scale = this.getScales(this.data)
      const scale1 = this.getScales(this.data1)
      const path = d3.line()
        .x((d, i) => scale.x(i))
        .y(d => scale.y(d))
      const path1 = d3.line()
        .x((d, i) => scale1.x(i))
        .y(d => scale1.y(d))
      console.log(d3.line().x((d, i) => scale1.x(i))
        .y(d => scale1.y(d))(this.data1))
      this.line = path(this.data)
      this.line1 = path1(this.data1)
    }
  }
}
</script>

<style scoped>
svg {
  margin: 25px;
}
path {
  fill: none;
  stroke: #76BF8A;
  stroke-width: 3px;
}
</style>
