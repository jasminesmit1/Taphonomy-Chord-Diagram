<template>
    <div class="chart">
        <h1>{{ msg }}</h1>
    </div>
</template>
<script>
    import * as d3 from 'd3'
    import dataset from '../assets/flare.json';
    // import {color} from "d3";
    // import {chord} from "d3";
    // import {arc} from "d3";
    // import {ribbon} from "d3";
    export default {
        name: 'chord',
        data: function(){
            return {
                msg: "Chord Diagram",
                width:954,
                height:954,
                outerRadius:477,
                innerRadius:353,
                data:[],
                dataset,
                name:["Ankle", "Arm", "Foot", "Hand", "Knee", "Leg",
                    "Pelvis", "Rib", "Shoulder","Sternum","Vertibral",
                    "Wrist"]
            }
        },
        mounted (){
            console.log("Chart Mounted");
            this.buildChart();
            // this.buildChart();
        },
        methods: {
            buildChart() {
                let vm = this;
                const imports = vm.dataset;
                let indexByName = new Map;
                let nameByIndex = new Map;
                let matrix = [];
                let n = 0;

                // Compute a unique index for each package name.
                imports.forEach(function(d) {
                    if (!indexByName.has(d = d.name)) {
                        nameByIndex.set(n, d);
                        indexByName.set(d, n++);
                    }
                });
                console.log("IBN: ", indexByName, " NBI: ", nameByIndex, " Matrix: ", matrix  )
                // Construct a square matrix counting package imports.
                imports.forEach(d => {
                    const source = indexByName.get(d.name);
                    let row = matrix[source];
                    if (!row) row = matrix[source] = Array.from({length: n}).fill(0);
                    Object.values(imports).forEach(d => {row[indexByName.get(d.name)]++});
                    // console.log(matrix)
                });


                const chord = d3.chord()
                    .padAngle(.04)
                    .sortSubgroups(d3.descending)
                    .sortChords(d3.descending)

                let  arc = d3.arc()
                    .innerRadius(this.innerRadius)
                    .outerRadius(this.innerRadius + 20)

                let ribbon = d3.ribbon()
                    .radius(this.innerRadius)

                 let color = d3.scaleOrdinal(d3.schemeCategory10)

                 let outerRadius = Math.min(this.width, this.height) * 0.5

                 this.innerRadius = outerRadius - 124


              d3.select(".chart").append("svg");
                const svg = d3.select("svg")
                    .attr("viewBox", [-this.width / 2, -this.height / 2, this.width, this.height])
                    .attr("font-size", 10)
                    .attr("font-family", "sans-serif")
                    .style("width", "100%")
                    .style("height", "auto");

                const chords = chord(matrix);

                const group = svg.append("g")
                    .selectAll("g")
                    .data(chords.groups)
                    .join("g");

                group.append("path")
                    .attr("fill", d => color(d.index))
                    .attr("stroke", d => color(d.index))
                    .attr("d", arc);

                group.append("text")
                    .each(d => {
                        d.angle = (d.startAngle + d.endAngle) / 2;
                    })
                    .attr("dy", ".35em")
                    .attr("transform", d => `rotate(${(d.angle * 180 / Math.PI - 90)})
                            translate(${this.innerRadius + 26})${d.angle > Math.PI ? "rotate(180)" : ""}
                          `)
                    .attr("text-anchor", d => d.angle > Math.PI ? "end" : null)
                    .text(function(d) {nameByIndex.get(d.index)});

                svg.append("g")
                    .attr("fill-opacity", 0.67)
                    .selectAll("path")
                    .data(chords)
                    .join("path")
                    .attr("stroke", d => d3.rgb(color(d.source.index)).darker())
                    .attr("fill", d => color(d.source.index))
                    .attr("d", ribbon);

            },

        }
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