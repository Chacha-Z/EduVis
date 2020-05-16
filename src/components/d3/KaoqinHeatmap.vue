<template>
  <div>
    <svg id='kaoqin-map' :width="width" :height="height" >
    </svg>
    <div class='kaoqin-tooltip' style='opacity: 0'></div>
  </div>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'kaoqin-map',
    props: ['student_id'],
    data(){
        return {
            data:{},
            margin: {top:60, right:0, bottom:0, left:50 },
            width: 460,
            height: 280,
        }
    },
    methods: {
        draw(){

            var dataset;
            var types = ["迟到", "早退", "校徽校服"],
                months = d3.range(1,13);
            
            // 定义长宽，以及热力图单位大小
            var w = this.width - this.margin.left - this.margin.right,
                gridSize = Math.floor(w / months.length),
                h = (gridSize/2) * (months.length+2);

            // 设置字体大小
            var newFontSize = w * 62.5 / 400;
            
            // 定义SVG container
            var svg = d3.select("#kaoqin-map")
                        .style("font-size", newFontSize + "%")
                        .append("g")
                            .attr("transform", "translate(" + this.margin.left + "," + this.margin.top + ")");
            
            var color = d3.scaleSequential(d3.interpolateBlues);
            
            var tooltip = d3.select(".kaoqin-tooltip");
            var dayLabels = svg.selectAll(".dayLabel")
                                .data(types)
                                .enter()
                                .append("text")
                                .text(function(d) { return d; })
                                .attr("x", 0)
                                .attr("y", function(d, i) { return i * 70+35; })
                                .style("text-anchor", "end")
                                .attr("transform", "translate(-6," + gridSize / 3 + ")")

            var timeLabels = svg.selectAll(".timeLabel")
                                .data(months)
                                .enter()
                                .append("text")
                                .text(function(d) { return d + '月'; })
                                .attr("x", function(d, i) { return i * gridSize; })
                                .attr("y", 0)
                                .style("text-anchor", "middle")
                                .attr("transform", "translate(" + gridSize / 2 + ", -6)");
                    
            var dataset = this.data;
            color.domain([-2, 4]);

            /* draw */

            
            var heatmap = svg.selectAll(".item")
                .data(dataset)
                .enter()
                .append("rect")
                .attr("x", function(d) { return (d.month-1) * gridSize; })
                .attr("y", function(d) { return types.indexOf(d.type) * 70; })
                .attr("class", "item")
                .attr("width", gridSize)
                .attr("height", 70)
                .style("stroke", "white")
                .style("stroke-opacity", 0.6)
                .style("fill", function(d) { return color(d.value); })
                .on("mousemove", function(d){
                    let coordinates = d3.mouse(this);
                    tooltip.style("left", (coordinates[0]+680)+'px')
                        .style("top", (coordinates[1]+5)+'px')
                        .style("display", "inline-block")
                        .html(d.type + "<br>" + (d.value))
                        .style("opacity", .9);
                })
                .on("mouseout", function(d){ tooltip.style("display", "none").style("opacity", 0);});
        },
        getData(){
            const request = axios.create({
                baseURL: 'http://localhost:3000'
            });

            request({
                url: '/stu/stuKaoqin',
                params: {
                    student_id: this.student_id
                }
            }).then(res => {
                this.data = res.data
                this.data.forEach(function(d) {
                    d.type = d.type;
                    d.month = +d.month;
                    d.value = +d.value;
                });
                this.draw();
            })
        }
    },
    created(){
        this.getData();
    },
    watch: { 
        student_id: function(){
            var content = d3.select('#kaoqin-map').selectAll('g').remove()
            // console.log('examview watch student_id: ', this.student_id);
            this.getData()
        }
    }
}

        
</script>

<style>
    #kaoqin-map {
      font-family: 'Source Sans Pro', sans-serif;
      fill: #696969;
      text-align: center;
    }
    
    .timeLabel, .dayLabel {
        fill: #AAAAAA;
    }

    .kaoqin-tooltip {
      position: absolute;
      width: 60px;
      height: 28px;
      font: 12px sans-serif;
      background: white;
      border: 0px;
      border-radius: 8px;
      padding: 2px;
      text-align: center;
    }
</style>