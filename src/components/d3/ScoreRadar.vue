<template>
    <svg id="score-radar" :width="width+marginX*2" :height="height+marginY*2">
    </svg>
    
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'stu-score-paralle',
    props: ['student_id'],
    data(){
        return {
            data: {},
            width: 120,
            height: 120,
            marginX: 50,
            marginY: 25,
            scales: {},
            xScale: null,
            axis: {}
        }
    },
    methods: {
        draw(){
            var cfg ={
                radius: 5,
                w: this.width,
                h: this.height,
                factor: 1,
                factorLegend: .85,
                levels: 2,
                maxValue: 0,
                minValue: 100,
                radians: 2 * Math.PI,
                opacityArea: 0.5,
                ToRight: 3,
                TranslateX: 0,
                TranslateY: 0,
                ExtraWidthX: this.marginX,
                ExtraWidthY: this.marginY,
                color: d3.scaleOrdinal().range(["#6F257F", "#CA0D59"])
            };
            
            cfg.color = d3.scaleOrdinal(d3.schemeCategory10);
            
            cfg.maxValue = Math.max(cfg.maxValue, d3.max(this.data, function(i){return d3.max(i.map(function(o){return o.value;}))}));
            cfg.minValue = Math.min(cfg.maxValue, d3.min(this.data, function(i){return d3.min(i.map(function(o){return o.value;}))}))-10;

            for(let i=0; i < this.data.length; ++i){
              for(let j=0; j < this.data[i].length; ++j){
                this.data[i][j].value -= cfg.minValue;
              }
            }
            cfg.maxValue -= cfg.minValue;

            var allAxis = (this.data[0].map(function(i, j){return i.axis}));
            var total = allAxis.length;
            var radius = cfg.factor*Math.min(cfg.w/2, cfg.h/2);
            
            var g = d3.select("#score-radar")
                        .append("g")
                        .attr("transform", "translate(" + cfg.TranslateX + "," + cfg.TranslateY + ")");

            var tooltip;
            
            //绘制内部等线
            for(var j=0; j<cfg.levels; j++){
                var levelFactor = cfg.factor*radius*((j+1)/cfg.levels);
                g.selectAll(".levels")
                    .data(allAxis)
                    .enter()
                    .append("svg:line")
                    .attr("x1", function(d, i){return levelFactor*(1-cfg.factor*Math.sin(i*cfg.radians/total));})
                    .attr("y1", function(d, i){return levelFactor*(1-cfg.factor*Math.cos(i*cfg.radians/total));})
                    .attr("x2", function(d, i){return levelFactor*(1-cfg.factor*Math.sin((i+1)*cfg.radians/total));})
                    .attr("y2", function(d, i){return levelFactor*(1-cfg.factor*Math.cos((i+1)*cfg.radians/total));})
                    .attr("class", "line")
                    .style("stroke", "grey")
                    .style("stroke-opacity", "0.75")
                    .style("stroke-width", "0.3px")
                    .attr("transform", "translate(" + (cfg.ExtraWidthX+cfg.w/2-levelFactor) + ", " + (cfg.ExtraWidthY+cfg.h/2-levelFactor) + ")");
            }

            var series = 0;

            var axis = g.selectAll(".axis")
                .data(allAxis)
                .enter()
                .append("g")
                .attr("class", "axis");

            axis.append("line")
                .attr("x1", cfg.ExtraWidthX + cfg.w/2)
                .attr("y1", cfg.ExtraWidthY + cfg.h/2)
                .attr("x2", function(d, i){return cfg.ExtraWidthX + cfg.w/2*(1-cfg.factor*Math.sin(i*cfg.radians/total));})
                .attr("y2", function(d, i){return cfg.ExtraWidthY + cfg.h/2*(1-cfg.factor*Math.cos(i*cfg.radians/total));})
                .attr("class", "line")
                .style("stroke", "grey")
                .style("stroke-width", "1px");

            axis.append("text")
                .attr("class", "legend")
                .text(function(d){return d})
                .style("font-family", "sans-serif")
                .style("font-size", "9px")
                .attr("text-anchor", "middle")
                .attr("dy", "1.5em")
                .attr("transform", function(d, i){return "translate(0, -10)"})
                .attr("x", function(d, i){return cfg.ExtraWidthX+cfg.w/2*(1-cfg.factorLegend*Math.sin(i*cfg.radians/total))-20*Math.sin(i*cfg.radians/total);})
                .attr("y", function(d, i){return cfg.ExtraWidthY+cfg.h/2*(1-Math.cos(i*cfg.radians/total))-10*Math.cos(i*cfg.radians/total);});
                // .attr("x", function(d, i){return cfg.w/2*(1-cfg.factor*Math.sin(i*cfg.radians/total));})
                // .attr("y", function(d, i){return cfg.h/2*(1-cfg.factor*Math.cos(i*cfg.radians/total));})

        
            this.data.forEach(function(y, x){
                var dataValues = [];
                g.selectAll(".nodes")
                .data(y, function(j, i){
                    dataValues.push([
                        cfg.ExtraWidthX+cfg.w/2*(1-(parseFloat(Math.max(j.value, 0))/cfg.maxValue)*cfg.factor*Math.sin(i*cfg.radians/total)), 
                        cfg.ExtraWidthY+cfg.h/2*(1-(parseFloat(Math.max(j.value, 0))/cfg.maxValue)*cfg.factor*Math.cos(i*cfg.radians/total))
                    ]);
                });
                dataValues.push(dataValues[0]);
                g.selectAll(".area")
                        .data([dataValues])
                        .enter()
                        .append("polygon")
                        .attr("class", "radar-chart-serie"+series)
                        .style("stroke-width", "2px")
                        .style("stroke", cfg.color(series))
                        .attr("points", function(d) {
                            var str="";
                            for(var pti=0;pti<d.length;pti++){
                                str=str+d[pti][0]+","+d[pti][1]+" ";
                            }
                            return str;
                        })
                        .style("fill", function(j, i){return cfg.color(series)})
                        .style("fill-opacity", cfg.opacityArea)
                        .on('mouseover', function (d){
                            var z = "polygon."+d3.select(this).attr("class");
                            g.selectAll("polygon")
                              .transition(200)
                              .style("fill-opacity", 0.1); 
                            g.selectAll(z)
                              .transition(200)
                              .style("fill-opacity", .7);
                            })
                        .on('mouseout', function(){
                            g.selectAll("polygon")
                            .transition(200)
                            .style("fill-opacity", cfg.opacityArea);
                        });
                series++;
            });
            series=0;

        },
        getData(){
          const request = axios.create({
            baseURL: 'http://localhost:3000'
          });

          request({
            url: '/stu/stuRadar',
            params: {
              student_id: this.student_id
            }
          }).then(res => {
            this.data = res.data
            // console.log(this.data);
            this.draw();
          })
        }
    },
    created(){
      this.getData();
    },
    watch: {
        student_id: function(){
          var content = d3.select('#score-radar').select('g');
          content.remove()
          this.getData();
        }
    }
}

</script>

<style scoped="scoped">

  .axis {
    font: 15px sans-serif;
  }
  .axis path,
  .axis line {
    fill: none;
    stroke: #D4D8DA;
    stroke-width: 2px;
    shape-rendering: crispEdges;
  }
  #chart {
    position: absolute;
    top: 50px;
    left: 100px;
  }
  
  .toolTip {
  pointer-events: none;
	position: absolute;
    display: none;
  min-width: 50px;
  height: auto;
  background: none repeat scroll 0 0 #ffffff;
  padding: 9px 14px 6px 14px;
  border-radius: 2px;
  text-align: center;
  line-height: 1.3;
  color: #5B6770;
  box-shadow: 0px 3px 9px rgba(0, 0, 0, .15);
  }
  .toolTip:after {
    content: "";
    width: 0;
    height: 0;
    border-left: 12px solid transparent;
    border-right: 12px solid transparent;
    border-top: 12px solid white;
    position: absolute;
    bottom: -10px;
    left: 50%;
    margin-left: -12px;
  }  
  .toolTip span {
    font-weight: 500;
    color: #081F2C;
  }

        
</style>