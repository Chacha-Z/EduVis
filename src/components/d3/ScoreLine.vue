<template>
  <div>
    <svg id='score-line' :width="width" :height="height">
    </svg>
    <div class='line-tooltip' style='display: none'></div>
  </div>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'stu-score-line',
    props: ['course_id', 'student_id', 'conceal_usual_performance'],
    data(){
        return {
            rawdata: [],
            margin: {
                top:40,
                bottom: 30,
                left: 50,
                right: 30
            },
            width: 470,
            height: 150,
        }
    },
    methods: {
        draw(){
            var raw_data = this.rawdata.filter(d => d.course_id == this.course_id); 
            
            if(this.conceal_usual_performance){
                raw_data = raw_data.filter(d => d.type_id != 4 && d.type_id != 22 && d.type_id != 5 && d.type_id != 18)
            }

            const $plot = d3.select('#score-line').append('g')
                .attr('transform', `translate(0, 0)`);
                // .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`);
            const svg = d3.select('#score-line')
            // prepare, set up scales
            const x = d3.scaleLinear();
            const y = d3.scaleLinear();

            const colour = d3.scaleOrdinal(d3.schemeCategory10);

            const xAxis = d3.axisBottom()
                            .scale(x)

            const yAxis = d3.axisLeft()
                            .scale(y)
                            .ticks(4);

            const line = d3.line()
                            .x(d => x(d.index))
                            .y(d => y(d.value))
                            .curve(d3.curveMonotoneX);

            const area = d3.area()
                            .x(d => x(d.index))
                            .y0((d, i) => y(d.min_score))
                            .y1((d, i) => y(d.max_score))
                            .curve(d3.curveMonotoneX);
            
            //binddata
            
            let xLength = raw_data.length;
            // 折线名
            const keys = ['score', 'average_score'];

            const data = keys.map(name => {
            return {
                name,
                values: raw_data.map((d,i) => {
                return {index:i, exam_name: d.name, value: +d[name]};
                })
            }
            });
            // 域名
            const areakeys = ['max_score', 'min_score']
            const areadata = raw_data.map((d,i) => {
                return {index:i, min_score: +d['min_score'], max_score: +d['max_score']};
            });

            // console.log('mapdata ', data);
            // console.log('areadata ', areadata);

            x.domain([0, xLength-1]);         

            y.domain([
                d3.min(areadata, c => c.min_score),
                d3.max(areadata, c => c.max_score)
            ]).nice();

            const $lines = $plot.append('g')
                            .attr('class', 'line')
                            .selectAll('.line')
                            .data(data)
                            .enter()
                            .append('g')
                            .attr('class', 'line')

            $lines.append('path')
                .attr('class', 'path');

            // console.log('lines', $lines);

            $plot.append('g')
                .attr('class', 'linearea')
                .append('path')
                .datum(areadata)
                .attr('class', 'backlinearea')

            $plot.append('g')
                .attr('class', 'axis x');

            $plot.append('g')
                .attr('class', 'axis y');
            
            //render
            
            const w = this.width- this.margin.left - this.margin.right;
            const h = this.height - this.margin.top - this.margin.bottom;

            x.range([this.margin.left, w]);
            y.range([h+this.margin.top, this.margin.top]);

            $plot.selectAll('.path')
                .attr('d', d => line(d.values))
                .attr('stroke', (d, i) => d.name == 'average_score' ? colour(1) : colour(0))
                .attr('opacity', d => d.name == 'score' ? 1 : 0.5)
                .attr('id', (d, i) => `line-${d.name}`)

            $plot.selectAll('.backlinearea')
                .attr('d', area)
                .attr('fill', 'grey')
                .attr('opacity', '0.12');
              
            $plot.select('.axis.x')
                .attr('transform', `translate(0, ${h+this.margin.top})`)
                .call(xAxis)

            $plot.select('.axis.y')
                .attr('transform', `translate(${this.margin.left}, 0)`)
                .call(yAxis)

            $plot.append('text')
                .attr('class', 'line-title')
                .text('分数趋势')
                .attr('transform', `translate(${this.margin.left - 10}, ${this.margin.top-10})`)

            $plot.append('rect')
                .attr('x', 0)
                .attr('y', this.margin.top)
                .attr('height', h)
                .attr('width', 1)
                .attr('class', 'ruler')
                .attr('opacity', 0)
                .attr('fill', 'grey')
            const $tooltip = d3.select('.line-tooltip');
            //鼠标移动事件
            svg.on('mousemove', function(d){
                const x_cood = d3.mouse(this)[0];
                const y_cood = d3.mouse(this)[1];
                const x_value = Math.round(x.invert(x_cood));
                if(x_value >= 0 && x_value < raw_data.length){

                    $plot.selectAll('.ruler')
                        .attr('x', x(x_value))
                        .attr('opacity', .7)

                    const chengji = raw_data[x_value];
                    const y_score = chengji.score, y_avgscore = chengji.average_score, 
                        y_minscore = chengji.min_score, y_maxscore = chengji.max_score;
                    const exam_title = chengji.name;
                
                    $tooltip.transition()
                        .duration(100)
                        .style('opacity', .7)
                        .style('display', 'block');
                    // console.log(x_cood + ', ' + y_cood)
                    $tooltip.html(exam_title + "<br/> <svg width='10', height='10'><circle cx='5', cy='5', r='5', style='fill:" + colour(0) + "'></svg> 分数: " + y_score 
                                    + "<br/><svg width='10', height='10'><circle cx='5', cy='5', r='5', style='fill:" + colour(1) + "'></svg> 平均分: " + y_avgscore 
                                    + "<br/><svg width='10', height='10'><circle cx='5', cy='5', r='5', style='fill: grey'></svg> 班级分数区间: [" + y_minscore +" : "+ y_maxscore + ']')
                    if(x_value < 7){
                        $tooltip.style('left', (x_cood+20)+'px')
                                .style('top', (y_cood-20)+'px');
                    }else{
                        $tooltip.style('left', (x_cood-200)+'px')
                                .style('top', (y_cood-20)+'px');
                    }
                }
            }).on('mouseout', function(){
                $plot.selectAll('.ruler')
                    .attr('opacity', 0)
                $tooltip.transition()		
                        .duration(200)		
                        .style('display', 'none');
            })
            .on('click', ()=>{

                const container = document.getElementById('score-line')
                const x_cood = d3.mouse(container)[0];
                const y_cood = d3.mouse(container)[1];
                const x_value = Math.round(x.invert(x_cood));
                if(x_value >= 0 && x_value < raw_data.length){

                    const chengji = raw_data[x_value];
                    // console.log(chengji)
                    this.clickExam(chengji);
                }
            })

            const legendtext = [{label:'分数', color: colour(0)}, {label:'平均分', color: colour(1)}, {label:'最高/低分', color: 'grey'}];
            //添加颜色标签
            const legend = $plot.append('g')
                            .attr('class', 'line-legend')
                            .attr("font-family", "sans-serif")
                            .attr("font-size", 10)
                            .attr('text-anchor', 'end')
                            .selectAll("g")
                            .data(legendtext)
                            .enter().append("g")
                            .attr("transform", (d, i) => `translate(${w+15}, ${i*20 +h/2})`);
            legend.append('rect')
                    .attr('width', 10)
                    .attr('height', 10)
                    .attr('opacity', .7)
                    .attr('fill', (d, i) => d.color)
                    .attr('rx', 2);
            legend.append('text')
                    .attr('x', 10)
                    .attr('dx', '0.325em')
                    .attr('y', 10)
                    .attr('dy', '-0.2em')
                    .attr('text-anchor', 'start')
                    .text(d => d.label)

        },
        getData(){
            const request = axios.create({
            baseURL: 'http://localhost:3000'
            });

            request({
                url: '/stu/stuCourseScore',
                params: {
                student_id: this.student_id
                }
            }).then(res => {
                this.rawdata = res.data
                // console.log(this.rawdata)
                this.draw();
            })
        },
        clickExam(d){
            this.$emit('exam-select', d.exam_id);
        }
    },
    created(){
        this.getData();
    },
    watch: {
        course_id: function(){
            var content = d3.select('#score-line').select('g');
            content.remove()
            // console.log('watch: ', this.course_id);
            this.draw()
        },
        student_id: function(){
            var content = d3.select('#score-line').selectAll('g').remove()
            // console.log('watch: ', this.student_id);
            this.getData()
        },
        conceal_usual_performance: function(){
            var content = d3.select('#score-line').select('g');
            content.remove()
            // console.log('watch: ', this.course_id);
            this.draw()
        },
    }
}

        
</script>

<style>

    #score-line {
      font-family: "avenir next", Arial, sans-serif;
      font-size: 12px;
    }
    .axis {
        font-size: 10px;
    }

    .axis path, .axis line {
      fill: none;
      stroke: #666;
      shape-rendering: crispEdges;
    }

    .line {
      fill: none;
      stroke-width: 1.5px;
    }

    .line-title {
        font-size: 12px;
        font-weight: bolder;
    }

    #line-Highlight {
      stroke-width: 2px;
    }

    .baseline {
      stroke: #666;
    }
                
    .line-tooltip {	
        position: absolute;			
        text-align: left;		
        padding: 10px;				
        font: 12px sans-serif;		
        color: snow;
        background: rgb(46, 50, 56);	
        border: 0px;		
        border-radius: 8px;
        pointer-events: none;
    }
</style>