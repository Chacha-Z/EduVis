<template>
  <div>
    <svg id='scatter-plot' :width="width" :height="height" >
    </svg>
    <div class='plot-tooltip' style='opacity: 0'></div>
    </div>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'stu-scatter-plot',
    props: ['class_id', 'course_id', 'student_id'],
    data(){
        return {
            data:{},
            margin: {
                top:30,
                bottom: 30,
                left: 35,
                right: 25
            },
            width: 350,
            height: 240,
        }
    },
    methods: {
        circleClick(d){
            // console.log('emit')
            this.$emit('stuselect', d.id);
        },
        draw(){
                        
            const curdata = this.data.filter(d => d.course_id == this.course_id); 

            const $plot = d3.select('#scatter-plot').append('g');

            const w = this.width- this.margin.left - this.margin.right;
            const h = this.height - this.margin.top - this.margin.bottom;

            const xScale = d3.scaleLinear();
            const yScale = d3.scaleLinear();

            const xAxis = d3.axisBottom(xScale);
            const yAxis = d3.axisLeft(yScale);
            
            
            const minstd = d3.min(curdata, d=>d.stdscore);
            const maxstd = d3.max(curdata, d=>d.stdscore);
            const minavg = d3.min(curdata, d=>d.avgscore);
            const maxavg = d3.max(curdata, d=>d.avgscore);

            xScale.domain([minstd-minstd*0.1, maxstd+minstd*0.1]);
            yScale.domain([minavg-minavg*0.1, maxavg+minavg*0.1]);

            const color = d3.scaleOrdinal(d3.schemeCategory10);

            $plot.append('g')
                    .attr('class', 'circles')
                    .selectAll('circle')
                    .data(curdata)
                    .enter()
                    .append('circle')
                    .attr('class', 'circle')
            
            $plot.append('g')
                    .attr('class', 'xaxis');
                    
            $plot.append('g')
                    .attr('class', 'yaxis');
                
            xScale.range([this.margin.left, w+this.margin.left]);
            yScale.range([h+this.margin.top, this.margin.top]);


            const $tooltip = d3.select('.plot-tooltip');
            $plot.selectAll('.circle')
                    .attr('cx', d=>xScale(d.stdscore))
                    .attr('cy',  d=>yScale(d.avgscore))
                    .attr('r', 5)
                    .style('fill', (d) => color(d.stusex))
                    .classed('nofocus', true)
                    .classed('stufocus', d=>d.id==this.student_id?true: false)
                    .on('mouseover', function(d){
                        d3.select(this)
                            .style('cursor', 'pointer')
                            .classed('mouseon', true)
                        $tooltip.transition()
                                .duration(100)
                                .style('opacity', .7)

                        let coordinates = d3.mouse(this);
                        if(coordinates[0] < 180){
                            $tooltip.html(d.stuname + '<br/>' + '平均分: ' + d.avgscore.toFixed(2) + '<br/>' + '标准差: ' + d.stdscore.toFixed(2))
                                .style('left', (coordinates[0]+10)+'px')
                                .style('top', (coordinates[1])+'px')
                        }else{
                            $tooltip.html(d.stuname + '<br/>' + '平均分: ' + d.avgscore.toFixed(2) + '<br/>' + '标准差: ' + d.stdscore.toFixed(2))
                                .style('left', (coordinates[0]-120)+'px')
                                .style('top', (coordinates[1])+'px')
                        }
                        
                    })					
                    .on("mouseout", function(d) {	
                        d3.select(this)
                            .classed('mouseon', false)
                        $tooltip.transition()		
                                .duration(200)		
                                .style("opacity", 0);	
                    })
                    .on('click', d => {
                        this.circleClick(d);
                    })
            
            $plot.select('.xaxis')
                .attr('transform', `translate(0, ${h+this.margin.top})`)
                .call(xAxis)
                .call(g => g.select(".domain")
                    .remove())
                .call(g => g.selectAll(".tick line")
                    .attr("stroke-opacity", 0.5)
                    .attr("stroke-dasharray", "2,2"))
                .call(g => g.selectAll(".tick text")
                    .attr("dx", 10)
                    .attr("dy", -3));
                
            $plot.select('.yaxis')
                .attr('transform', `translate(${this.margin.left}, 0)`)
                .call(yAxis)
                .call(g => g.select(".domain")
                    .remove())
                .call(g => g.selectAll(".tick line")
                // .call(g => g.selectAll(".tick:not(:first-of-type) line")
                    .attr("stroke-opacity", 0.5)
                    .attr("stroke-dasharray", "2,2"))
                .call(g => g.selectAll(".tick text")
                    .attr("x", 4)
                    .attr("dy", -4))

            //添加坐标轴标签
            $plot.append('g')
                .attr('class', 'axis-title')
                .append('text')
                .attr('transform', `translate(${this.margin.left+w/2}, ${h+this.margin.top+22})`)
                .style('text-anchor', 'middle')
                .text('标准差')
            $plot.append('g')
                .attr('class', 'axis-title')
                .attr('transform', `translate(${this.margin.left-20}, ${this.margin.top+h/2})`)
                .append('text')
                .attr('transform', 'rotate(-90)')
                // .attr('x', this.margin.left-25)
                // .attr('y',  this.margin.top+h/2)
                .style('text-anchor', 'middle')
                .text('平均分')

            const keys = ['女','男'];
            //添加颜色标签
            const legend = $plot.append('g')
                    .attr('class', 'plot-legend')
                    .attr("font-family", "sans-serif")
                    .attr("font-size", 10)
                    .attr('text-anchor', 'end')
                    .selectAll("g")
                    .data(keys)
                    .enter().append("g")
                    .attr("transform", (d, i) => `translate(${w+this.margin.left}, ${i*20 +h/2})`);
            legend.append('rect')
                    .attr('width', 10)
                    .attr('height', 10)
                    .attr('opacity', .7)
                    .attr('fill', (d, i) => color(i))
                    .attr('rx', 2);
            legend.append('text')
                    .attr('x', 10)
                    .attr('dx', '0.325em')
                    .attr('y', 10)
                    .attr('dy', '-0.2em')
                    .attr('text-anchor', 'start')
                    .text(d => d)
        },
        getData(){
            const request = axios.create({
                baseURL: 'http://localhost:3000'
            });

            request({
                url: '/stu/stuAllPlot',
                params: {
                course_id: this.course_id,
                class_id: this.class_id
                }
            }).then(res => {
                this.data = res.data
                this.draw();
            })
        }
    },
    created(){
        this.getData();
    },
    watch: {
        course_id: function(){
            var content = d3.select('#scatter-plot').selectAll('g');
            content.remove()
            // console.log('watch-plot: ', this.course_id);
            this.draw()
        },
        class_id: function(){
            // console.log('plot class change', this.class_id)
            var content = d3.select('#scatter-plot').selectAll('g');
            content.remove()
            this.getData();
        },
        student_id: function(){
            //修改样式
            // console.log('plot stu change', this.student_id)
            var stucircles = d3.selectAll('circle');
            // console.log(stucircles)
            stucircles.classed('nofocus', true).
                        classed('stufocus',  d=>d.id==this.student_id?true: false);
        }
    }
}

        
</script>

<style>
    .axis-title {
        font-size: 10px;
        letter-spacing: 5px;        
    }
    .plot-tooltip {	
        position: absolute;			
        text-align: center;		
        height: 50px;
        width: 90px;			
        padding: 10px;				
        font: 12px sans-serif;		
        color: snow;
        background: rgb(46, 50, 56);	
        border: 0px;		
        border-radius: 8px;			
        pointer-events: none;
    }
    .nofocus {
        fill-opacity: 0.4;
    }
    .stufocus {
        fill-opacity: 1;
    }
    .mouseon {
        fill-opacity: 1;
    }
</style>