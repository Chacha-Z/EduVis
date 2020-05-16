<template>
    <div>
        <svg :id="'bar'+student_id"></svg>
    </div>
</template>

<script>
import * as d3 from 'd3'
import axios from 'axios'
export default {
    name: 'stu-overall_bar',
    props: ['student_id', 'rawdata'],
    data(){
        return {
            width: 180,
            height: 30,
            old_stuid: 0
        }
    },
    methods: {
        draw(){
                
            const overall_data = this.rawdata.filter(d => d.stuid == this.student_id)

            // console.log(overall_data);

            var result = {}
            for(let i = 0; i < overall_data.length; ++i){
                result[overall_data[i].courseid] = overall_data[i].score;
            }

            const $plot = d3.select('#bar'+this.old_stuid)
                            .attr('width', this.width)
                            .attr('height', this.height)
                            .append('g');
            var stack = d3.stack()
                            .keys(["1", "2", "3", "4", "5", "6", "7", "8", "17"])
                            .order(d3.stackOrderNone)
                            .offset(d3.stackOffsetNone);

            const series = stack([result]);
            // console.log(series);

            const color = d3.scaleOrdinal(d3.schemeCategory10);
            const unit = this.width/900;

            $plot.selectAll('g')
                .data(series)
                .enter().append('g')
                .attr('fill', (d, i) => color(d.key))
                .style('opacity', '.9')
                .selectAll('rect')
                .data(d => d)
                .enter().append('rect')
                .attr('mydata', d=>d)
                .attr('width', (d, i) => unit*(d[1]-d[0]))
                .attr('height', 25)
                .attr('x', (d, i) => unit*d[0])
                .attr('y', 8)
        }
    },
    created(){
        this.old_stuid = this.student_id;
    },
    mounted(){
        this.draw()
    },
    watch: {
        student_id: function(){
            var content = d3.select('#bar'+this.old_stuid).selectAll('g');
            content.remove()
            this.draw()
            this.old_stuid = this.student_id;
        }
    }
}

        
</script>

<style>
</style>