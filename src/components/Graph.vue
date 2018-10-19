<template>
  <div class="hello">
    <div class="graph-wrap">
      <div class="graph row" id="graph"></div>
    </div>
  </div>
</template>

<script>
import gCrosshair from '../lib/drawGraph.js';

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

<style lang="scss">
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
.graph-wrap {
  background: #273652;
  padding: 20px;
}
.graph {
  font-size: 10px;
  // margin-top: 50px;
}
path.candle {
  stroke: #yellow;
}
path.candle.body {
  stroke-width: 0;
}
path.candle.up {
  fill: #27DC8D;
  stroke: #27DC8D;
}
path.candle.wick.up {
  fill: red;
}
path.candle.down {
  fill: #FF2752;
  stroke: #FF2752;
}
text.coords {
  stroke: white;
}
.crosshair {
  cursor: crosshair;
}
.crosshair path.wire {
  stroke: #af4c4c;
  stroke-dasharray: 1, 1;
}
.crosshair .axisannotation path {
  fill: #dddddd;
}

g.x.axis {
  stroke: white;
}
path.domain {
  stroke: white;
}

g.tick {
  stroke: white;
  line: {
    stroke: white;
  }
}

.companyName {
  stroke: white;
}
</style>
