<template>
    <el-table 
        ref="singleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 100%"
        height='100%' 
        
        :row-class-name="tableRowClassName"
        highlight-current-row
        @current-change="handleCurrentChange">
            <!-- <el-table-column type="selection" width="55"></el-table-column> -->
            <el-table-column prop="stuid" label="学号" sortable></el-table-column>
            <el-table-column prop="stuname" label="姓名" sortable ></el-table-column>
            <el-table-column prop="id" label="概览" width='190' >
              <template slot="header" slot-scope="scope">
                <div>
                  <span>状态</span>
                  <el-tooltip class="tooltip" placement="right">
                    <i class="el-icon-question" style="color:#409eff;margin-left:5px;font-size:15px;"></i>
                    <div slot='content' style='text-align: center; font: 12px sans-serif'>
                      以T分数平均分为基础<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#1f77b4'/></svg> 语文<br/> 
                      <svg width='10' height='10'><rect width='10' height='10' fill='#ff7f0e'/></svg> 数学<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#2ca02c'/></svg> 英语<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#d62728'/></svg> 物理<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#9467bd'/></svg> 化学<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#8c564b'/></svg> 生物<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#e377c2'/></svg> 历史<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#7f7f7f'/></svg> 地理<br/>
                      <svg width='10' height='10'><rect width='10' height='10' fill='#bcbd22'/></svg> 政治<br/>
                    </div>
                  </el-tooltip>
            
                </div>
              </template>
              <template slot-scope="scope">
                <overall-bar :student_id="scope.row.stuid" :rawdata="overallData"></overall-bar>
              </template>
            </el-table-column>
    </el-table>
    <!-- <div style="margin-top: 20px">
        <el-button @click="toggleSelection([tableData[1], tableData[2]])">切换第二、第三行的选中状态</el-button>
        <el-button @click="toggleSelection()">取消选择</el-button>
    </div> -->
</template>

<script>
import axios from 'axios'
import OverallBar from './d3/OverallBar'
  export default {
    props: ['class_id', 'student_id'],
    data() {
      return {
        tableData: [],
        overallData: [],
        currentRow: null
      }
    },
    components: {
      'overall-bar': OverallBar,
    },
    created(){
      this.getData();
    },
    methods: {
      setCurrent(row) {
        this.$refs.singleTable.setCurrentRow(row);
        var index = row.index;
        const targetTop = this.$refs.singleTable.$el.querySelectorAll('.el-table__body tr')[index].getBoundingClientRect().top
        const containerTop = this.$refs.singleTable.$el.querySelector('.el-table__body').getBoundingClientRect().top
        const scrollParent = this.$refs.singleTable.$el.querySelector('.el-table__body-wrapper')
        // console.log('top:', targetTop)
        // console.log('container:',containerTop)
        // console.log('scroll:', targetTop - containerTop)
        scrollParent.scrollTop = targetTop - containerTop
      },
      handleCurrentChange(val) {
        this.currentRow = val;
        this.$emit('stuchange', val.stuid);
      },
      tableRowClassName ({row, rowIndex}) {
        //把每一行的索引放进row
        row.index = rowIndex;
      },
      getData(){
        const request = axios.create({
          baseURL: 'http://localhost:3000'
        });

        axios.all([axios.get('http://localhost:3000/stu/selectClassStu',{
          params: {
            class_id: this.class_id
          }
        }),
        axios.get('http://localhost:3000/stu/selectClassoverall', {
          params: {
            class_id: this.class_id
          }
        })
        ]).then(axios.spread((res1, res2) => {
          this.tableData = res1.data;
          this.overallData = res2.data;
          // console.log('table: ', this.tableData);
          // console.log('over:', this.overallData)
        }))
      },
    },
    watch: {
        class_id: function(){
            // console.log('table class change', this.class_id)
            this.getData()
        },
        student_id: function(){
            // console.log('table stu change', this.student_id) 
            var tarrow = this.tableData.filter(d=>d.stuid == this.student_id)[0];
            // console.log(tarrow)
            if(tarrow != undefined)
              this.setCurrent(tarrow)
        }
    }
  }
</script>

<style>
  .el-table__header-wrapper .el-table__header th {
    padding: 5px 0;
  }


  .el-table__body .el-table__row:hover{
    cursor: pointer;
  }
  .el-table__body .el-table__row td{
    padding: 0 0;
    height: 40px;
  }
</style>