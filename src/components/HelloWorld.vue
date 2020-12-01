<template>
  <line :data="animatedData">
    <template v-slot="lastHoverPoints"></template>
  </line>
</template>

<script>
import { line } from './line'
import moment from 'moment'
export default {
  name: 'HelloWold',
  components: {
    line
  },
  data () {
    return {
      animatedData: []
    }
  },
  computed: {
    padded () {
      const width = this.width - this.margin.left - this.margin.right
      const height = this.height - this.margin.top - this.margin.bottom
      return { width, height }
    }
  },

  watch: {},
  methods: {},

  mounted () {
    this.animatedData = []
    // let color = ['green', 'blue', 'yellow', 'orange', 'red'];
    let color = ['#7cb55d', '#5d72b5', '#d9ca64', '#b5785d', '#5daeb5']
    let name = [
      '生活リズム',
      '課題遂行率',
      '健康感',
      'コミュニケーション評価',
      '抑うつ・不安'
    ];
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
            color: color[lineIndex],
            id: lineIndex,
            seriesName: name[lineIndex]
          }
        }
        if (lineIndex === 1 && index % 7 < 5) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex],
            id: lineIndex,
            seriesName: name[lineIndex]
          }
        }

        if (lineIndex === 2 && index % 7 < 2) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex],
            id: lineIndex,
            seriesName: name[lineIndex]
          }
        }

        if (lineIndex === 3 && index % 7 < 1) {
          this.animatedData[lineIndex][this.animatedData[lineIndex].length] = {
            date: moment('2020-12-25')
              .startOf('day')
              .add(index, 'day')
              .format('YYYY-MM-DD'),
            value: Math.ceil(Math.random() * 100),
            color: color[lineIndex],
            id: lineIndex,
            seriesName: name[lineIndex]
          }
        }
      })
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.chart {
  height: 100%;
  width: 100%;
  -webkit-tap-highlight-color: transparent;
  user-select: none;
  position: relative;
}
.tooltip {
  position: absolute;
}
</style>
