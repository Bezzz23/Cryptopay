<template>
  <div class="hello">
    <div class="graph row" id="graph"></div>
  </div>
</template>

<script>
import gCrosshair from '../lib/g-crosshair.js';

 export default {
  props: ["stock", "graph"],
  data() {
    return {
      width: 0
    };
  },
  watch: {
    graph() {
      //if the graph data is updated, rerender the graph
      if (this.graph.length) {
        this.redrawGraph();
      } else {
        this.cleanupGraph();
      }
    }
  },
  //run when component is mounted (add resize event)
  mounted() {
    window.addEventListener("resize", this.redrawGraph);
  },
  //run when component is destroyed (remove resize event)
  beforeDestroy() {
    window.removeEventListener("resize", this.redrawGraph);
  },
  methods: {
    //computes chart dimensions based on page width
    computeDimensions() {
      var graphEl = document.getElementById("graph");
      this.width = graphEl.offsetWidth;
    },
    //removes current chart
    cleanupGraph() {
      document.getElementById("graph").innerHTML = "";
    },
    //redraw the chart with new dimensions
    redrawGraph() {
      this.cleanupGraph();
      this.computeDimensions();
      var x = this.graph.length - 1;
      var y = this.graph[x].open;
      gCrosshair(
        this.graph,
        this.stock.companyName,
        this.width,
        x,
        y
      );
    }
  }
};
</script>

<style scoped lang="scss">
.quote {
  margin-top: 50px;
}
.quote .row {
  padding: 10px;
}
.quote .row div:first-child {
  font-weight: 500;
}
.quote .row:nth-child(odd) {
  background-color: #f0f0f0;
}
.graph {
  font-size: 10px;
  margin-top: 50px;
}
path.candle {
  stroke: #000000;
}
path.candle.body {
  stroke-width: 0;
}
path.candle.up {
  fill: #00aa00;
  stroke: #00aa00;
}
path.candle.down {
  fill: #ff0000;
  stroke: #ff0000;
}
.crosshair {
  cursor: crosshair;
}
.crosshair path.wire {
  stroke: #dddddd;
  stroke-dasharray: 1, 1;
}
.crosshair .axisannotation path {
  fill: #dddddd;
}
</style>
