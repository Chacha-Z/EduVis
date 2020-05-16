<template>
    <div>
        <svg id='score-paralle' :width="width" :height="height">
        </svg>
        <div class='parallel-tooltip' style='display: none'></div>
    </div>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'stu-score-paralle',
    props: ['student_id', 'conceal_usual_performance'],
    data(){
        return {
            chengji: {},
            margin: {left: 40, right: 50, top: 50, bottom: 30},
            width: 640,
            height: 310,
            columns: [
                {label: "语文", accessor: d => d['1']},
                {label: "数学", accessor: d => d['2']},
                {label: "英语", accessor: d => d['3']},
                {label: "物理", accessor: d => d['4']},
                {label: "化学", accessor: d => d['5']},
                {label: "生物", accessor: d => d['6']},
                {label: "历史", accessor: d => d['7']},
                {label: "地理", accessor: d => d['8']},
                {label: "政治", accessor: d => d['17']}
            ],
            scales: {},
            xScale: null,
            axis: {}
        }
    },
    methods: {
        initial(){
            var dataset = this.chengji;
            if(this.conceal_usual_performance){
                dataset = this.chengji.filter(d => d['-3'] != 4 && d['-3'] != 22 && d['-3'] != 5 && d['-3'] != 18)
            }
            this.xScale = d3.scaleLinear()
                            .domain([0, this.columns.length - 1])
                            .range([this.margin.left, this.width-this.margin.right]);

            for (const i in this.columns) {
                var column = this.columns[i];
                column.x = this.xScale(i);
                this.scales[column.label] = d3.scaleLinear().range([this.margin.top, this.height-this.margin.bottom]);
                this.axis[column.label] = d3.axisLeft(this.scales[column.label]);
            }

            var svg = d3.select("#score-paralle")
                        .append("g");

            var maxScale = 0;
            for (let i in this.columns) {
                const column = this.columns[i];

                let tempValue = d3.max(dataset, column.accessor);
                if(tempValue> maxScale){
                    maxScale = tempValue;
                }

                // 坐标轴标签
                svg.append("g")
                    .append("text")
                    .attr("class", "label")
                    .attr("x", column.x)
                    .attr("y", this.margin.top-20)
                    .attr("font-size", 12)
                    .attr("font-style", "normal")
                    .style("text-anchor", "middle")
                    .style('dominant-baseline', 'text-before-edge')
                    .text(column.label)
                    .call(g => g.selectAll("text")   
                            .clone(true).lower()
                            .attr("fill", "none")
                            .attr("stroke-width", 5)
                            .attr("stroke-linejoin", "round")
                            .attr("stroke", "white"));;

            }

            for(let i in this.columns){
                const column = this.columns[i];
                this.scales[column.label].domain([1, maxScale]).clamp();
            }

            //计算path数组
            var paths = [];
            for (var i in dataset) {
                const row = dataset[i];
                var pathPoints = [];
                for (var j in this.columns) {
                    const column = this.columns[j];
                    const y = this.scales[column.label](column.accessor(row));
                    if (isNaN(y)) continue;
                    pathPoints.push({
                        x: column.x, 
                        y: y,
                        label: column.label, 
                        value: column.accessor(row),
                        title: row['-1'],
                        exam_id: row['-2']
                    });
                }
                paths.push(pathPoints);
                // console.log(row);
            }
            
            const color = d3.scaleOrdinal(d3.schemeCategory10);
            const farestcolor = color(0);
            const latestcolor = color(1);//'#2ca02c', '#d62728'
            let colorLinear = d3.scaleLinear().domain([0, paths.length]).range([0, 1]);
            let ColorCompute = d3.interpolate(latestcolor, farestcolor);

            const $tooltip = d3.select('.parallel-tooltip');


            // 画线
            svg.append("g")
            /*.append("path")
            .datum(paths)
            */
                .selectAll("path")
                .data(paths)
                .enter()
                .append("path")
                .attr('class', 'parallel-path')
                .attr("fill", "none")
                .attr("stroke", (d, i) => ColorCompute(colorLinear(i)))
                .attr("stroke-width", 1)
                .attr("d", d3.line().x(d => d.x).y(d => d.y))
                .on('mouseover', function(d){
                    d3.selectAll('.parallel-path')
                        .attr("opacity", .2);
                    d3.select(this)
                        .attr('stroke-width', 3)
                        .attr("opacity", 1);
                    $tooltip.transition()
                            .duration(100)
                            .style('opacity', .7)
                            .style('display', 'block');
                    let coordinates = d3.mouse(this);

                    let htmltext = d[0].title + '<br/>';
                    for(let i = 0; i < d.length; ++i){
                        if(i != d.length-1){
                            htmltext += d[i].label + ': ' + d[i].value + '<br/>'
                        }else{
                            htmltext += d[i].label + ': ' + d[i].value;
                        }
                    }

                    $tooltip.html(htmltext)
                            .style('left', (coordinates[0]+10)+'px')
                            .style('top', coordinates[1]-d.length*10+'px')
                            // .style('left', '20px')
                            // .style('top', '30px')
                })
                .on('mouseout', function(d){
                    d3.select(this)
                        .attr('stroke-width', 1);
                    d3.selectAll('.parallel-path')
                        .attr("stroke", (d, i) => ColorCompute(colorLinear(i)))
                        .attr("opacity", 1);
                    $tooltip.transition()		
                            .duration(200)		
                            .style('display', 'none');
                })
                .on('click', (d)=>{
                    this.lineClick(d);
                })

            // 最后画出坐标轴（避免覆盖）
            for (var i in this.columns) {
                const column = this.columns[i];

                // 坐标轴
                svg.append("g")
                .attr("class", "y axis")
                .attr("transform", "translate(" + column.x + ", 0)")
                .call(this.axis[column.label])
                .call(g => g.selectAll("text")   //添加坐标轴背景
                            .clone(true).lower()
                            .attr("fill", "none")
                            .attr("stroke-width", 5)
                            .attr("stroke-linejoin", "round")
                            .attr("stroke", "white"));
            }

            const gradientcolorset = [{color: ColorCompute(colorLinear(0)), offset: '0%'}, {color: ColorCompute(colorLinear(paths.length)), offset: '100%'}];
            //添加渐变颜色标签
            svg.append('defs')
                .append('linearGradient')
                .attr('id', 'linear-gradient')
                .attr('x1', '0%')
                .attr('x2', '0%')
                .attr('y1', '0%')
                .attr('y2', '100%')
                .selectAll('stop')
                .data(gradientcolorset)
                .enter().append('stop')
                .attr('offset', d => d.offset)
                .attr('stop-color', d => d.color)
            
            svg.append('g')
                .attr('id', 'color-legend')
                .attr('transform', `translate(${this.width-this.margin.left+15}, ${this.margin.top+20})`)
                .append('rect')
                .attr('width', 18)
                .attr('height', 190)
                .attr('fill', 'url(#linear-gradient)')
                .attr('rx', 2)

            svg.select('#color-legend')
                .attr("font-family", "sans-serif")
                .attr("font-size", 10)
                .attr('text-anchor', 'start')
                .selectAll('text')
                .data(['最近', '最久'])
                .enter().append('text')
                .attr('x', 0)
                .attr('y', (d, i) => 190*i - 3)
                .attr('dy', (d, i) => 1.3*i + 'em')
                .text(d => d)
                
        },
        getData(){
            
            const request = axios.create({
                baseURL: 'http://localhost:3000'
            });
            request({
                url: '/stu/stuOrder',
                params: {
                    student_id: this.student_id
                }

            }).then(res => {

                this.chengji = res.data
                // console.log(this.chengji)
                this.initial();
            }).catch(err => console.log(err))
        },
        lineClick(d){
            // console.log(d)
            this.$emit('exam-select', d[0].exam_id);
        }
    },
    created(){
        this.getData();
    },
    watch: {
        student_id: function(){
            var content = d3.select('#score-paralle').select('g');
            content.remove()
            // console.log('watch: ', this.course_id);
            this.getData()
        },
        conceal_usual_performance: function(){
            var content = d3.select('#score-paralle').select('g');
            content.remove()
            this.initial();
        }
    }
}

        
</script>

<style>

    .axis path,
    .axis line {
        fill: none;
        stroke: #000;
        shape-rendering: crispEdges;
    }

    .dot {
        stroke: #000;
    }

    text {
        font-family: Helvetica;
    }

    /* path:hover {
        stroke: goldenrod;
        opacity: 1;
        stroke-width: 2;
        pointer-events:all;
    } */
        
    .parallel-tooltip {	
        position: absolute;			
        text-align: center;		
        padding: 10px;				
        font: 12px sans-serif;		
        color: snow;
        background: rgb(46, 50, 56);	
        border: 0px;		
        border-radius: 8px;			
        pointer-events: none;
        z-index: 1000000;
    }
</style>