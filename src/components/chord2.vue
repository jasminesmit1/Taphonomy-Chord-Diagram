<template>
    <div class="chart">
        <h1>{{ msg }}</h1>
        <div class="container" id="chart1">
        </div>
        <div class="container" id="chart2">
        </div>
    </div>
</template>

<script>
    import * as d3 from 'd3'
    import data from '../assets/chartdata2.json';
    // import {color} from "d3";
    // import {chord} from "d3";
    // import {arc} from "d3";
    // import {ribbon} from "d3";
    export default {
        name: "chord2",
        data: function(){
            return {
                msg: " Taphonomy Chord Diagram",
                width: 954,
                height: 954,
                outerRadius: 477,
                innerRadius: 353,
                selected_accession: "",
                selected_prov1: "",
                groups: [],
                data,
                categories:[],
                taphonomies: {},
                sampledData:[],
                matrix:[],
                groupIndexes: new Set,
                specimens: [],

            }
        },
        mounted (){
            console.log("Chart Mounted");
            this.getSampleData();
            this.getCategories();
            this.formatData();
            // this.buildMatrix();
            this.buildChart();
        },
        methods: {
            getCategories() { /* dynamically creates categories based off bone_category in data.
                            Used to grouping on chord diagram*/
                let categories = new Set();
                this.sampledData.forEach(d => {
                    categories.add(d.bone_category)
                })
                this.categories = Array.from(categories);
            },
            getSampleData() { /* Original dataset too large. This function randomly samples the dataset*/
                let i = 12;
                while (i--) {
                    let index = Math.floor(Math.random() * this.data.length);
                    this.sampledData.push(this.data[index])
                }
            },
            formatData() { /* formats sampled data to be used in chord diagram*/
                this.sampledData.forEach((z) => {
                    let group = {}; /*creates new a group object each iteration to append to this.group*/
                    let se = [];/*stores each specimen composite key and their taphonomy id
                                as a list within group object*/
                    group.category = z.bone_category;
                    let specimen = z.composite_key;
                    let taphonomy = z.taphonomy_id;
                    se.push(specimen, taphonomy);
                    group.specimens = se;
                    this.groups.push(group);
                })
                this.groups.sort((a,b) => a.category.localeCompare(b.category))
                // console.log("groups " ,this.groups)

                let taph_ids = d3.nest() /* groups specimens by their taphonomy id in a json*/
                    .key(d => {
                        return d.specimens[1];
                    })
                    .entries(this.groups);
                this.taphonomies = taph_ids;
                for (let obj of this.groups){
                    let k  = [];
                    k.push(obj.specimens[0], obj.specimens[1])
                    this.specimens.push(k)
                }

                // console.log("taph: ", this.taphonomies)
            },
            buildChart() {
                /* constructs an adjaceny matrix for each specimen in the sample set
             '1' if they have a taphomomy in common and '0' if they do not.*/
                let n = this.groups.length;
                // let matrix = new Object();
                let matrix = new Array(n).fill(0);
                for (let i = 0; i < matrix.length; i++)
                    matrix[i] = new Array(n).fill(0);
                let list = Object.keys(this.groups).map(val => this.groups[val].category);
                let indexlist =[];
                matrix.forEach((d) => {
                    this.groups.forEach((z) => {
                        // console.log(z.specimens[0])
                        let groupIndexes ={};
                        let sIndex = list.findIndex((c) => c === z.category);
                        let eIndex = list.lastIndexOf(z.category);
                        let category = z.category; /* ADD TO LIST IF CATEGORY ALREADY EXIST DONT ADD TO GROUPINDEXES!!!!*/
                        let n = indexlist.indexOf(category)
                        if(n === -1) {
                            indexlist.push(category);
                            groupIndexes.sIndex = sIndex;
                            groupIndexes.eIndex = eIndex;
                            groupIndexes.category = category;
                            this.groupIndexes.add(groupIndexes)
                        }
                        this.groups.slice(1).forEach((t, r) => {
                            // console.log(z.specimens[0],z.specimens[1],t.specimens[0],t.specimens[1]);
                            if (z.specimens[0] !== t.specimens[0] && z.specimens[1] === t.specimens[1]) {
                                d[r] = 1;
                            }
                        })
                    })
                })

                let GroupColors = ["#FF6666", "#99FF99", "#66FFFF", "#A0A0A0",
                    "#ff9933", "#9933FF", "#99004C", "#440154ff","#6b6ecfff",
                    "#b5cf6bff", "#e7ba52ff", "#4287f5","#d6616bff"]

                let groupfill = d3.scaleOrdinal()
                    .domain(d3.range(this.categories.keys()))
                    .range(GroupColors);

                var chord_layout = d3.chord()
                    .padAngle(0.03)
                    .sortSubgroups(d3.descending);

                var width = 800;
                var height = 800;
                var innerRadius = width / 2 * 0.7;
                var outerRadius = innerRadius * 1.1;

                var tooltip = d3.select("#chart2")
                    .append("div")
                    .style("opacity", 0)
                    .attr("class", "tooltip")
                    .style("background-color", "white")
                    .style("border", "solid")
                    .style("border-width", "1px")
                    .style("border-radius", "5px")
                    .style("padding", "10px")

                //add element
                var svg = d3.select("#chart1")
                    .append("svg")
                    .attr("viewBox", "-20 40 900 900")
                    .attr("width", width)
                    .attr("height", height);


                //draw nodes
                var outer_arc = d3.arc()
                    .innerRadius(innerRadius)
                    .outerRadius(outerRadius);

                var chord = chord_layout(matrix);

                var g_outer = svg.append("g")
                    .attr("transform", "translate(" + width / 2 + "," + height / 2 + ")")
                    .datum(chord);

                var group = g_outer.append("g")
                    .attr("class", "groups")
                    .selectAll("g")
                    .data(function(chords) { return chords.groups; })
                    .enter().append("g");

                let se1 = this.specimens;
                 // console.log(se1)


                //add color
                group.append("g").attr("class", 'se')
                    .append("path")
                    .style("fill", function(d) {
                        return groupfill(d.index);
                    })
                    .style("stroke", function(d) {
                        return groupfill(d.index);
                    })
                    .attr("d", outer_arc)
                    .attr("id",function(d,i){return "group"+i;});

                 group.selectAll("g")
                    .append("text")
                    .attr("x", 2)
                    .attr("dy", 15)
                    .attr("font-size", "8px")
                    .append("textPath")
                    .attr("class", "label")
                    .attr("xlink:href", function(d) { return "#group" + d.index; })
                    .text(function(d) {
                                return se1[d.index][0];})
                    .style("fill", "#030303")
                    .style("stroke", "#030303");
                //add chord
                var inner_chord = d3.ribbon()
                    .radius(innerRadius);
                // var taph = this.taphonomies;

                g_outer.append("g")
                    .attr("class", "ribbons")
                    .selectAll("path")
                    .data(function(chords) { return chords; })
                    .enter().append("path")
                    .attr("d", inner_chord)
                    .style("fill", function(d)
                    { return groupfill(d.source.index);})
                    .style("stroke", "black")
                    .style("opacity", 0.6)
                    .on("mouseover", function(d) {
                        var pageX=d3.event.pageX;
                        var pageY=d3.event.pageY;
                        // console.log(taph, se1[d.source.index][0]);
                        d3.select(this)
                            .style("fill", "crimson");
                        tooltip
                            .style("opacity", 1)
                            .html("Source: " + se1[d.source.index][0] + "<br> Target: " + se1[d.target.index][0]
                                + "<br>Taphonomony Id: " + se1[d.source.index][1])
                            .style("left", (pageX + 15) + "px")
                            .style("top", (pageY - 28) + "px");
                    })
                    .on("mouseout", function(d) {
                        d3.select(this)
                            .transition()
                            .duration(1000)
                            .style("fill", groupfill(d.source.index));
                        // tooltip
                        //     .transition()
                        //     .duration(1000)
                        //     .style("opacity", 0);
                    });

                group.append("g")
                    .attr("class", "ribbons")
                    .selectAll("path")
                    .data(function(chords) { return chords; })
                    .enter().append("path")
                    .attr("d", inner_chord)
                    .style("fill", function(d) { return groupfill(d.target.index); })
                    .style("stroke", function(d) { return d3.rgb(groupfill(d.target.index)).darker(); });

                let index = Array.from(this.groupIndexes);
                let labelg = svg.append("g")
                var cD = chord_layout(matrix).groups
                // console.log(cD, index)
                //draw arcs
                for(var i=0;i<index.length;i++) {
                    var __g = index[i];
                    var arc1 = d3.arc()
                        .innerRadius(innerRadius*1.11)
                        .outerRadius(outerRadius*1.1)
                        .startAngle(cD[__g.sIndex].startAngle)
                        .endAngle(cD[__g.eIndex].endAngle)

                    labelg.attr("transform", "translate(" + width / 2 + "," + height / 2 + ")")
                        .append("path").attr("d", arc1).attr('id', 'groupId' + i);

                    // Add a text label.
                    var text = labelg.append("text")
                        .attr("x", 65)
                        .attr("dy", 20)
                        // .attr("font-size", "10px")
                        .style("text-anchor", "middle");


                    text.append("textPath")
                        .attr("stroke","#fff")
                        .attr('fill', '#fff')
                        .attr("xlink:href","#groupId" + i)
                        .text(__g.category);
                }
            },

        }
}
</script>

<style scoped>
    .container {
        float: left;
    }

</style>