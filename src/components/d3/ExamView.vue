<template>
    <svg id='exam-view' :width="width" :height="height">
    </svg>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'stu-exam-view',
    props: ['exam_id', 'student_id'],
    data(){
        return {
            rawdata: [],
            margin: {
                top:40,
                bottom: 10,
                left: 60,
                right: 10
            },
            width: 630,
            height: 350,
        }
    },
    methods: {
        draw(){
            // console.log(this.rawdata);
            const data = this.rawdata.filter(d => d.exam_id == this.exam_id)
            // console.log(data);
            const keys = ['score', 'class_rank', 'z_score', 't_score', 'dengdi_score']

            const keys_name = ['分数', '班级排名', 'Z分数', 'T分数', '等第成绩']

            const keys_values = [];
            const min_max_values = [];
            for(let i = 0; i < keys.length; ++i){
                let item = data.map(item => {
                    return {
                        key: keys[i], 
                        value: item[keys[i]]
                    }
                });

                keys_values.push(item);
                if(keys[i] == 'score'){
                    min_max_values[keys[i]] = {
                            min_value: d3.min(item, d=>d.value<0?null:d.value)-30,
                            max_value: d3.max(item, d=>d.value<0|d.value>150?null:d.value)+10
                        }
                }else if(keys[i] == 'class_rank'){
                    min_max_values[keys[i]] = {
                            min_value: d3.min(item, d=>d.value<0?null:d.value)-15,
                            max_value: d3.max(item, d=>d.value)+5
                        }
                }else if(keys[i] == 't_score'){
                    min_max_values[keys[i]] = {
                            min_value: d3.min(item, d=>d.value<0?null:d.value)-10,
                            max_value: d3.max(item, d=>d.value)+5
                        }
                }else if(keys[i] == 'z_score'){
                    min_max_values[keys[i]] = {
                            min_value: d3.min(item, d=>d.value)-3,
                            max_value: d3.max(item, d=>d.value)+1
                        }
                }else{
                    min_max_values[keys[i]] = {
                            min_value: d3.min(item, d=>d.value)-0.2,
                            max_value: d3.max(item, d=>d.value)+0.2
                        }
                }
                
            }

            // console.log(keys_values);
            // console.log(min_max_values);

            const colors = {'score':d3.scaleSequential(d3.interpolateBlues).domain([min_max_values.score.min_value, min_max_values.score.max_value]),
                            'class_rank': d3.scaleSequential(d3.interpolateGreens).domain([min_max_values.class_rank.min_value, min_max_values.class_rank.max_value]),
                            'z_score': d3.scaleSequential(d3.interpolateOranges).domain([min_max_values.z_score.min_value, min_max_values.z_score.max_value]),
                            't_score': d3.scaleSequential(d3.interpolatePurples).domain([min_max_values.t_score.min_value, min_max_values.t_score.max_value]),
                            'dengdi_score': d3.scaleSequential(d3.interpolateReds).domain([min_max_values.dengdi_score.min_value, min_max_values.dengdi_score.max_value]),
                            }
            const svg = d3.select('#exam-view')
            const $plot = svg.append('g')
                            .attr('transform',`translate(${this.margin.left}, ${this.margin.top})`)

            const xScale = d3.scaleBand().domain(d3.range(data.length)).range([0, this.width-this.margin.left-this.margin.right]).padding(0.1);
            const yScale = d3.scaleBand().domain(d3.range(keys.length)).range([0, this.height-this.margin.top-this.margin.bottom]).padding(0.5);

            const keysg = $plot.selectAll('g')
                            .data(keys_values)  //行
                            .enter().append('g')
                            .attr('transform', (d, i)=>`translate(0, `+yScale(i) +')')
                            
            const rectg = keysg.selectAll('g')
                .data(d => d)        //列
                .enter().append('g')    
            rectg.append('rect')
                .attr('width', d => xScale.bandwidth())
                .attr('height', yScale.bandwidth())
                .attr('x', (d, i)=> xScale(i))
                .attr('fill', d => d.key=='score'?(d.value<0?'lightgrey':colors[d.key](d.value)):d.value==null?'lightgrey':colors[d.key](d.value));
            rectg.append('text')
                .attr('class', 'score-text')
                .text(d=>d.key=='score'?(d.value==-1?'作弊': d.value==-2?'缺考':d.value==-3?'免考':Math.floor(d.value * 100) / 100):d.value==null?'':Math.floor(d.value * 100) / 100)
                .attr('x', (d, i)=>xScale(i)+xScale.bandwidth()/2)
                .attr('dy', yScale.bandwidth()/2)
                .style('text-anchor', 'middle')
                .style('dominant-baseline', 'middle')
                .attr('fill', 'white')
                .style('font-size', 12)

            $plot.append('g')
                .attr('class', 'course-legend')
                .selectAll('text')
                .data(data)
                .enter().append('text')
                .text(d => d.course_name)
                .attr('x', (d, i)=> xScale(i)+xScale.bandwidth()/2)
                .style('text-anchor', 'middle')
                .style('font-size', 12)
            
            $plot.append('g')
                .attr('class', 'score-legend')
                // .attr('transform', `translate(0, 0)`)
                .selectAll('text')
                .data(keys_name)
                .enter().append('text')
                .text(d => d)
                .attr('y', (d, i)=> yScale(i)+yScale.bandwidth()/2)
                .style('text-anchor', 'end')
                .style('dominant-baseline', 'middle')
                .style('font-size', 12)
        },
        getData(){
            const request = axios.create({
                baseURL: 'http://localhost:3000'
            });

            request({
                url: '/stu/selectExam',
                params: {
                student_id: this.student_id,
                }
            }).then(res => {
                this.rawdata = res.data
                // console.log(this.rawdata)
                var content = d3.select('#exam-view').selectAll('g').remove()
                this.draw()
            })
        }
    },
    created(){
        this.getData();
    }, 
    watch: {  
        exam_id: function(){
            var content = d3.select('#exam-view').selectAll('g').remove()
            // console.log('examview watch exam_id: ', this.exam_id);
            this.draw()
        },
        student_id: function(){
            // var content = d3.select('#exam-view').selectAll('g').remove()
            // console.log('examview watch student_id: ', this.student_id);
            this.getData()
        }
    }
}

        
</script>

<style>
</style>