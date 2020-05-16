<template>
  <div>
    <svg id='consumption-line' :width='width' :height='height'>
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'consumption-line',
    props: ['student_id'],
    data(){
        return {
            rawdata:{},
            margin: { top: 50, right: 70, bottom: 80, left: 35 },
            width: 520,
            height: 180,
        }
    },
    methods: {
        init(){
            const $svg = d3.select('#consumption-line')
            const $plot = $svg.append('g')
                            .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`);

            $svg.attr('width', this.width + this.margin.left + this.margin.right)
                .attr('height', this.height + this.margin.top + this.margin.bottom);

            // 初始化比例尺
            const x = d3.scaleTime().range([0, this.width]);
            const y = d3.scaleLinear().range([this.height, 0]);

            const colour = d3.scaleOrdinal(d3.schemeCategory10);

            const xAxis = d3.axisBottom(x);

            const yAxis = d3.axisLeft(y);

            const margin2 = { top: this.height + this.margin.top + 30, right: this.margin.right, bottom: 30, left: this.margin.left };
            const height2 = this.height + this.margin.top + this.margin.bottom -margin2.top-margin2.bottom;
            const x2 = d3.scaleTime().range([0, this.width]);
            const y2 = d3.scaleLinear().range([height2, 0]);
            const xAxis2 = d3.axisBottom(x2);
            const line2 = d3.line().x(d => x2(d.date)).y(d => y2(d.cost));
            const $context = $svg.append('g')
                                .attr('class', 'context')
                                .attr('transform', `translate(${margin2.left}, ${margin2.top})`);

            const brush = d3.brushX()
                            .extent([[0, 0], [this.width, height2]])
                            .on('brush', ()=>{
                                const selection = d3.event.selection;
                                var dateRange = selection.map(x2.invert, x2);
                                x.domain(dateRange);
                                line.x(d => x(d.date))
                                    .y(d => y(d.cost));
                                    
                                $plot.selectAll('.path')
                                        .attr('d', line)
                                        .attr('stroke', (d, i) => colour(i))
                                
                                $plot.selectAll('.line-label')
                                    .attr('transform', d => {
                                        return `translate(${x(d.value.date)}, ${y(d.value.cost)})`;
                                    });
                                
                                $plot.select('.axis.x').call(xAxis);
                            })
                            .on('end', ()=>{
                                
                                const selection = d3.event.selection;


                                console.log(selection);
                                console.log(selection.map(x2.invert, x2));

                                var dateRange = selection.map(x2.invert, x2);
                                x.domain(dateRange);
                                $plot.select('.axis.x').call(xAxis);
                            });
            const line = d3.line()
                            .x(d => x(d.date))
                            .y(d => y(d.cost))
                            .curve(d3.curveMonotoneX);

            const area = d3.area()
                        .x(d => d)
                        .y0((d, i) => y(6))
                        .y1((d, i) => y(12))

            /* binddata*/
            var data = this.rawdata;
            const keys = ['cost', 'avgcost'];

            var parseDate = d3.timeParse('%Y-%m-%d');
            data.forEach(element => {
                element.forEach(d=>{
                    return d.date = parseDate(d.date);
                })
            });

            x.domain([d3.min(data, set=>d3.min(set, d=>d.date)), d3.max(data, set=>d3.max(set, d=>d.date))]);         
            
            y.domain([
                d3.min(data, set => d3.min(set, d=>d.cost)),
                d3.max(data, set => d3.max(set, d=>d.cost))
            ]).nice();

            x2.domain(x.domain());
            y2.domain(y.domain());

            const $lines = $plot.append('g')
                            .attr('class', 'lines')
                            .selectAll('.line')
                            .data(data)
                            .enter()
                            .append('g')
                            .attr('class', 'line')

            $lines.append('path')
                    .attr('class', 'path');

            $lines.append('text')
                .datum((d, i) => {
                    return {
                        name: keys[i],
                        value: d[d.length-1]
                    }
                })
                .attr('class', 'line-label')
                .attr('opacity', 0)


            $plot.append('g')
                .attr('class', 'axis x');

            $plot.append('g')
                .attr('class', 'axis y');

            const $area = $plot.append('g')
                        .attr('class', 'area')
                        .append('path')
                        .datum(d3.range(0, this.width))
                        .attr('class', 'backarea');
            $plot.select('.area')
                    .append('text')
                    .attr('class', 'area-label')
                    .attr('opacity', 0);

            $context.append('path')
                    .attr('class', 'context_path')
                    .datum(data[0]);
            $context.append('g')
                    .attr('class', 'context axis');
            
            /* render */

            $plot.select('.axis.x')
                .attr('transform', `translate(0, ${this.height})`)
                .call(xAxis);

            $plot.select('.axis.y')
                .attr('transform', 'translate(0, 0)')
                .call(yAxis);
            $plot.append('text')
                    .attr('x', 0)
                    .attr('y', -5)
                    .style('text-anchor', 'start')
                    .text('元')

            $svg.append('clipPath')
                .attr('id', 'line-area')
                .append('rect')
                .attr('x', 0)
                .attr('y', 0)
                .attr('width', this.width)
                .attr('height', this.height);

            $plot.select('.lines')
                    .attr('clip-path', 'url(#line-area)');

            $plot.selectAll('.path')
                    .attr('d', line)
                    .attr('stroke', (d, i) => colour(i))
            

            $plot.selectAll('.line-label')
                .attr('transform', d => {
                    return `translate(${x(d.value.date)}, ${y(d.value.cost)})`;
                })
                .attr('x', 5)
                .attr('dy', '.35em')
                .attr('fill', (d,i) => colour(i))
                .attr('font-weight', d => d.name == 'cost' ? 700 : 400)
                .text(d => d.name)
                .attr('opacity', 1);
            
            $plot.selectAll('.area-label')
                    .attr('x', this.width)
                    .attr('y', y(6))
                    .attr('fill', 'grey')
                    .attr('text-anchor', 'end')
                    .attr('dominant-baseline', 'bottom')
                    .text('合理消费区间')
                    .attr('opacity', 0.75);

            
            $plot.selectAll('.backarea')
                    .attr('d', area)
                    .attr('fill', 'grey')
                    .attr('opacity', '0.25');
            
            $svg.select('.context_path')
                .attr('d', line2)
                .attr('stroke', 'grey');
            $svg.select('.context.axis')
                .attr('transform', `translate(0, ${height2})`)
                .call(xAxis2);
            $context.append('g')
                .call(brush);
                // .call(brush.move, x2.range());

            //坐标轴标签
            const legends = ['消费', '平均'];
            //添加颜色标签
            const legend = $plot.append('g')
                    .attr('class', 'plot-legend')
                    .attr("font-family", "sans-serif")
                    .attr("font-size", 10)
                    .attr('text-anchor', 'end')
                    .selectAll("g")
                    .data(legends)
                    .enter().append("g")
                    .attr("transform", (d, i) => `translate(${this.width+this.margin.left}, ${i*20 +this.height/2})`);
            legend.append('rect')
                    .attr('width', 10)
                    .attr('height', 10)
                    .attr('opacity', .7)
                    .attr('fill', (d, i) => colour(i))
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
                url: '/stu/stuConsumption',
                params: {
                  student_id: this.student_id,
                }
            }).then(res => {
                this.rawdata = res.data
                this.init();
            })
        }
    },
    created(){
        this.getData();
    },
    watch: { 
        student_id: function(){
            var content = d3.select('#consumption-line').selectAll('g').remove()
            // console.log('examview watch student_id: ', this.student_id);
            this.getData()
        }
    }
}

        
</script>

<style>
    #consumption-line { 
      font-family: "avenir next", Arial, sans-serif;
      font-size: 12px;
      color: #666;
    }

    #consumption-line .axis path, #consumption-line .axis line {
      fill: none;
      stroke: #666;
      shape-rendering: crispEdges;
    }

    #consumption-line .line {
      fill: none;
      stroke-width: 1.5px;
    }

    #consumption-line .context_path {
      fill: none;
      stroke-width: 1px;
    }


    #line-Highlight {
      stroke-width: 2px;
    }

    #consumption-line .baseline {
      stroke: #666;
    }
</style>