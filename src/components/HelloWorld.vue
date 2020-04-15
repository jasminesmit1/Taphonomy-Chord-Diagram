<template>
  <div class="chart">
    <h1>{{ msg }}</h1>
  </div>
</template>
<script>
import * as d3 from 'd3'
import data from '../assets/chartdata.json';
// import {color} from "d3";
// import {chord} from "d3";
// import {arc} from "d3";
// import {ribbon} from "d3";
export default {
  name: 'HelloWorld',
  data: function(){
    return {
      msg: "Chord Diagram",
      width:954,
      height:954,
      outerRadius:477,
      innerRadius:353,
      chart: {},
      data,
      name:["Ankle", "Arm", "Foot", "Hand", "Knee", "Leg",
        "Pelvis", "Rib", "Shoulder","Sternum","Vertibral",
        "Wrist"]
    }
  },
  mounted (){
    console.log("Chart Mounted");
    this.buildChart();
    // this.gatherData();
    // this.buildChart();
  },
  methods: {
    buildChart() {
      var colors = ["#6b6ecfff","#b5cf6bff", "#e7ba52ff", "#d6616bff",
        "#ce6dbdff","#9c9edeff", "#440154ff", "#cedb9cff",
        "#8B4513", "#34eb89", ]

      // var tooltip = d3.select(".chart")
      //         .append("div")
      //         .style("opacity", 0)
      //         .attr("class", "tooltip")
      //         .style("background-color", "white")
      //         .style("border", "solid")
      //         .style("border-width", "1px")
      //         .style("border-radius", "5px")
      //         .style("padding", "10px")

      var svg = d3.select(".chart")
              .append("svg")
              .attr("width", 540)
              .attr("height", 440)
              .append("g")
              .attr("transform", "translate(295,220)")

      var chord = d3.chord()
              .padAngle(0.05)
              .sortSubgroups(d3.descending)(this.data.matrix)

      svg.datum(chord)
              .append("g")
              .selectAll("g")
              .data(function(d) { return d.groups; })
              .enter()
              .append("g")
              .append("path")
              .style("fill", function(d,i){ return colors[i] })
              .style("stroke", "black")
              .attr("d", d3.arc()
                      .innerRadius(200)
                      .outerRadius(210))

      svg.datum(chord)
              .append("g")
              .selectAll("path")
              .data(function(d) { return d; })
              .enter()
              .append("path")
              .attr("d", d3.ribbon()
                      .radius(200)
              )
              .style("fill", function(d){ return(colors[d.source.index]) }) // colors depend on the source group. Change to target otherwise.
              .style("stroke", "black")
              // .on("mouseover", showTooltip )
              // .on("mouseleave", hideTooltip );

      var group = svg
              .datum(chord)
              .append("g")
              .selectAll("g")
              .data(function(d) { return d.groups; })
              .enter()

      group.selectAll(".group-tick")
              .data(function(d) { return groupTicks(d,1); })
              .enter()
              .append("g")
              .attr("transform", function(d) { return "rotate(" + (d.angle * 180 / Math.PI - 90) + ") translate(" + 200 + ",0)"; })
              .append("line")               // By default, x1 = y1 = y2 = 0, so no need to specify it.
              .attr("x2", 6)
              .attr("stroke", "black")

      group.selectAll(".group-tick-label")
              .data(function(d) { return groupTicks(d,250); })
              .enter()
              .append("g")
              .attr("transform", function(d) { return "rotate(" + (d.angle * 180 / Math.PI - 90) + ") translate(" + 200 + ",0)"; })
              .append("text")
              .attr("x", 8)
              .attr("dy", ".35em")
              .attr("transform", function(d) { return d.angle > Math.PI ? "rotate(180) translate(-16)" : null; })
              .style("text-anchor", function(d) { return d.angle > Math.PI ? "end" : null; })
              .text(function(d) {return d.value})
              .style("font-size", 9)



      function groupTicks(d, step) {
        var k = (d.endAngle - d.startAngle) / d.value;
        return d3.range(0, d.value, step).map(function (value) {
          return {value: value, angle: value * k + d.startAngle};
        });
      }
      // var showTooltip = function(d) {
      //   tooltip
      //           .style("opacity", 1)
      //           .html("Source: " + this.name[d.source.index] + "<br>Target: " + this.name[d.target.index])
      //           .style("left", (d3.event.pageX + 15) + "px")
      //           .style("top", (d3.event.pageY - 28) + "px")
      // }

      // var hideTooltip = function(d) {
      //   tooltip
      //           .transition()
      //           .duration(1000)
      //           .style("opacity", 0)
      // }


    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
