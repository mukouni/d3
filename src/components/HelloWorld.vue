<template>
  <div>
    dddd
    <svg @mousemove="mouseover" :width="width" :height="height" id="d3-svg">
      <g :style="{transform: `translate(${margin.left}px, ${margin.top}px)`}">
        <path class="area" :d="paths.area" />
        <template v-for="line in paths.lines">
          <path class="line" :d="paths.line" />
        </template>
        <path class="selector" :d="paths.selector" />
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from "d3";
import TWEEN from "tween.js";
export default {
  name: "HelloWorld",
  props: {
    data: {
      type: Array,
      default: () => [100, 2, 13, 41, 5, 16, 71, 18, 19, 10]
    },
    margin: {
      type: Object,
      default: () => ({
        left: 0,
        right: 0,
        top: 10,
        bottom: 10
      })
    },
    ceil: {
      type: Number,
      default: 100
    }
  },
  data() {
    return {
      width: 0,
      height: 200,
      paths: {
        area: "",
        line: "",
        selector: ""
      },
      lastHoverPoint: {},
      scaled: {
        x: null,
        y: null
      },
      animatedData: [],
      points: []
    };
  },
  computed: {
    padded() {
      const width = this.width - this.margin.left - this.margin.right;
      const height = this.height - this.margin.top - this.margin.bottom;
      return { width, height };
    }
  },
  mounted() {
    this.animatedData = [];
    [...new Array(5)].forEach((_, lineIndex) => {
      if (!this.animatedData[lineIndex]) this.animatedData[lineIndex] = [];
      [...new Array(3 * 7)].forEach((_, index) => {
        this.animatedData[lineIndex][index] = {
          date: moment().add(index, "day"),
          value: Math.ceil(Math.random() * 100)
        };
      });
    });
    window.addEventListener("resize", this.onResize);
    this.onResize();
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onResize);
  },
  watch: {
    data: function dataChanged(newData, oldData) {
      console.log(newData);
      // this.tweenData(newData, oldData);
    },
    width: function widthChanged(data) {
      console.log(data);
      this.tweenData();
      this.initialize();
      this.update();
    }
  },
  methods: {
    x: d3
      .scaleTime()
      .domain(d3.extent([100, 2, 13, 41, 5, 16, 71, 18, 19, 10], (d, i) => i))
      .range([0, this.width]),
    onResize() {
      this.width = this.$el.offsetWidth;
      this.height = this.$el.offsetHeight;
    },
    createArea: d3
      .area()
      .x(d => d.x)
      .y0(d => d.max)
      .y1(d => d.y),
    // createArea: d3
    //   .area()
    //   .x(d => {
    //     console.log(d.x);
    //     return d.x;
    //   })
    //   .y0(d => {
    //     console.log(d.max);
    //     return d.max;
    //   })
    //   .y1(d => {
    //     console.log(d.y);
    //     return 0;
    //   }),
    createLine: d3
      .line()
      .x(d => {
        return this.x(d.x);
      })
      .y(d => d.y),
    createValueSelector: d3
      .area()
      .x(d => d.x)
      .y0(d => d.max)
      .y1(0),
    initialize() {
      this.scaled.x = d3.scaleLinear().range([0, this.padded.width]);
      console.log(this.scaled.x, d3);
      this.scaled.y = d3.scaleLinear().range([this.padded.height, 0]);
      d3.axisLeft().scale(this.scaled.x);
      d3.axisBottom().scale(this.scaled.y);
    },
    tweenData(newData, oldData) {
      const vm = this;
      function animate(time) {
        requestAnimationFrame(animate);
        TWEEN.update(time);
      }
      console.log(newData, oldData, TWEEN);
      // new TWEEN.Tween(oldData)
      //   .easing(TWEEN.Easing.Quadratic.Out)
      //   .to(newData, 500)
      //   .onUpdate(function onUpdate() {
      //     vm.animatedData = this;
      //     vm.update();
      //   })
      //   .start();
      animate();
    },
    update() {
      this.scaled.x.domain(d3.extent(this.data, (d, i) => i));
      this.scaled.y.domain([0, this.ceil]);
      this.points = [];
      for (const [index, data] of this.animatedData.entries()) {
        this.points.push({
          x: this.scaled.x(index),
          y: this.scaled.y(data),
          max: this.height
        });
      }

      // console.log(this.createArea(this.points));
      // this.paths.area = this.createArea(this.points);
      this.paths.line = this.createLine(this.points);
    },
    mouseover({ offsetX }) {
      if (this.points.length > 0) {
        const x = offsetX - this.margin.left;
        const closestPoint = this.getClosestPoint(x);
        if (this.lastHoverPoint.index !== closestPoint.index) {
          const point = this.points[closestPoint.index];
          this.paths.selector = this.createValueSelector([point]);
          this.$emit("select", this.data[closestPoint.index]);
          this.lastHoverPoint = closestPoint;
        }
      }
    },
    getClosestPoint(x) {
      return this.points
        .map((point, index) => ({
          x: point.x,
          diff: Math.abs(point.x - x),
          index
        }))
        .reduce((memo, val) => (memo.diff < val.diff ? memo : val));
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
