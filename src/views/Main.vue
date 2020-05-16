<template>
    <main>
      <el-row class='topnavi'>
        <top-navi :class_id=classid v-on:class-change="classchange"></top-navi>
      </el-row>
      <el-row :gutter="3" class='outer'>
        <el-col :span="6" class='overall'>
          <el-row style="height: 270px;">
            <div class="grid-content bg-purple-light" ><!-- 散点图 -->
              <title-bar>班级散点图<select-box style="float: right; margin-right: 5px" v-on:select-change="plotchange"></select-box></title-bar>
              <scatter-plot :student_id='studentid' :course_id="plotcourseid" :class_id="classid" v-on:stuselect="stuchange"></scatter-plot>
            </div>
          </el-row>
          
          <el-row style="height: 430px;"><!-- 学生列表 -->
              <stu-table :student_id='studentid' :class_id="classid" v-on:stuchange="stuchange"></stu-table>
          </el-row>
        </el-col>
        <el-col :span="18" class="detail">
          <el-row :gutter="0">
              <el-col :span="14" style="height: 360px;">
                <div class="grid-content bg-purple-light">
                  <title-bar>考试详情<exam-box :student_id="studentid" :exam_value="examid" v-on:exam-change='examchange' style="float: right; margin-right: 5px" ></exam-box></title-bar><!-- 最近一次考试成绩分析 -->
                  <exam-view :student_id="studentid" :exam_id="examid"></exam-view>
                </div>
              </el-col>
              <el-col :span="10">
                <el-row>
                  <el-col style="height: 170px;"> <!-- 个人详细信息 -->
                    <div class="grid-content bg-purple-light">
                      <title-bar>个人信息</title-bar>
                      <detail-table :student_id="studentid" style="height: 150px;"></detail-table>
                    </div>
                  </el-col>
                </el-row>
                <el-row >
                  <el-col :span="12" style="height: 190px;">
                    <div class="grid-content bg-purple-light"><!-- 雷达图 -->
                      <title-bar>学生能力雷达图 
                        <el-tooltip class="tooltip" placement="right">
                          <i class="el-icon-question" style="color:#409eff;margin-left:5px;font-size:15px;"></i>
                          <div slot='content' style='text-align: center; font: 12px sans-serif'>
                            以T分数平均分为基础<br/>
                          </div>
                        </el-tooltip>
                      </title-bar>
                      <score-radar :student_id="studentid"></score-radar>
                    </div>
                  </el-col>
                  <el-col :span="12"  style="height: 190px;"><!-- 词云 -->
                    <div class="grid-content bg-purple-light">
                      <title-bar>词云</title-bar>
                      <word-cloud :student_id="studentid"></word-cloud>
                    </div>
                  </el-col>
                </el-row>
              </el-col>
          </el-row>
          <el-row :gutter="0">
              <el-col :span="14" style="height: 340px;">
                <div class="grid-content bg-purple-light"><!-- 平行坐标轴 -->
                  <title-bar>所有考试排名概览<usual-switch style="float: right; margin-right: 25px" :conceal_usual_performance="conceal_usual_performance" v-on:conceal-change="concealchange"></usual-switch></title-bar>
                  <paralle-lines :student_id="studentid" v-on:exam-select="examchange"  :conceal_usual_performance="conceal_usual_performance" ></paralle-lines>
                </div>
              </el-col>
              <el-col :span="10">
                <el-row style="height: 170px;">
                  <div class="grid-content bg-purple-light"><!-- 折线图 -->
                    <title-bar>成绩趋势<select-box style="float: right; margin-right: 5px" v-on:select-change="linechange"></select-box></title-bar>
                    <score-line :course_id="linecourseid" :student_id="studentid" v-on:exam-select="examchange" :conceal_usual_performance="conceal_usual_performance" ></score-line>
                  </div>
                </el-row>
                <el-row style="height: 170px;">
                  <div class="grid-content bg-purple-light"><!-- 折线图 -->
                    <rank-line :course_id="linecourseid" :student_id="studentid" v-on:exam-select="examchange" :conceal_usual_performance="conceal_usual_performance" ></rank-line>
                  </div>
                </el-row>
              </el-col>
          </el-row>
          <el-row :gutter="0">
              <el-col :span="14" style="height: 340px;">
                <div class="grid-content bg-purple-light"><!-- 折线图 -->
                  <title-bar>消费数据</title-bar>
                  <consumption-line :student_id="studentid"></consumption-line>
                </div>
              </el-col>
              <el-col :span="10" style="height: 340px;">
                <div class="grid-content bg-purple-light"><!-- 热力图 -->
                  <title-bar>考勤数据</title-bar>
                  <kaoqin-map :student_id="studentid"></kaoqin-map>
                </div>
              </el-col>
          </el-row>
        </el-col>
      </el-row>
    </main>
</template>

<script>
import axios from 'axios'
import StuTable from '../components/StuTable'
import ParalleLines from '../components/d3/ParalleLines'
import ScoreRadar from '../components/d3/ScoreRadar'
import ScatterPlot from '../components/d3/ScatterPlot'
import ScoreLine from '../components/d3/ScoreLine'
import RankLine from '../components/d3/RankLine'
import TitleBar from '../components/TitleBar'
import SelectBox from '../components/SelectBox'
import ExamBox from '../components/ExamBox'
import DetailTable from '../components/DetailTable'
import ExamView from '../components/d3/ExamView'
import WordCloud from '../components/WordCloud'
import TopNavigator from '../components/TopNavigator'
import ConsumptionLine from '../components/d3/ConsumptionLine'
import KaoqinMap from '../components/d3/KaoqinHeatmap'
import Switch from '../components/Switch'
export default {
    data() {
      return {
        classid: 0,
        studentid: 14454,
        plotcourseid: 0,
        linecourseid: 0,

        examid: 0,
        conceal_usual_performance: false
      }
    },
    components: {
        'stu-table': StuTable,
        'paralle-lines': ParalleLines,
        'score-radar': ScoreRadar,
        'scatter-plot': ScatterPlot,
        'score-line': ScoreLine,
        'rank-line': RankLine,
        'title-bar': TitleBar,
        'select-box': SelectBox,
        'exam-box': ExamBox,
        'detail-table': DetailTable,
        'exam-view': ExamView,
        'word-cloud': WordCloud,
        'top-navi': TopNavigator,
        'consumption-line': ConsumptionLine,
        'kaoqin-map': KaoqinMap,
        'usual-switch': Switch
    },
    methods: {
      linechange(courseid){
        console.log(courseid);
        this.linecourseid = courseid;
      },
      plotchange(courseid){
        console.log(courseid);
        this.plotcourseid = courseid;
      },
      examchange(examid){
        console.log(examid);
        this.examid = examid;
      },
      stuchange(stuid){
        this.studentid = stuid;
        // console.log('stuchange', stuid)
      },
      classchange(classid){
        console.log(classid)
        this.classid = classid
      },
      concealchange(value){
        console.log(value)
        this.conceal_usual_performance=value;
      }
    },
    watch: {
      classid: function(){
        const request = axios.create({
          baseURL: 'http://localhost:3000'
        });

        request({
          url: '/stu/selectClassStu',
          params: {
            class_id: this.classid
          }
        }).then(res => {
          this.studentid = res.data[0].stuid;
        })
        
      }
    }

}
</script>

<style>
  body {
      padding: 0px;
      margin: 0px;
      background-color: #e5e9f2;
  }

  main {
      padding: 3px 5px 0 5px;
      min-width: 1420px;
  }

  .el-row.topnavi {
    margin-bottom: 3px;
    /* height: 300px; */
  }

  .overall {
    height: 700px;
    /* overflow: auto; */
  }
  .detail {
    height: 708px;
    overflow: auto;
    -ms-overflow-style: none;    
    overflow: -moz-scrollbars-none; 
  }
  .detail::-webkit-scrollbar { width: 0 !important }

  /* .el-row:last-child {
    margin-bottom: 0;
  } */
  .el-col {
    border-radius: 2px;
  }
  .bg-purple-dark {
    background: #99a9bf;
  }
  .bg-purple {
    background: #d3dce6;
  }
  .bg-purple-light {
    background: #ffffff;
  }
  .grid-content {
    border-radius: 2px;
    height: 100%;
    width: 100%;
  }
  .row-bg {
    padding: 5px 0;
    background-color: #f9fafc;
  }
</style>