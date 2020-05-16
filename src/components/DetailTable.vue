<template>
  <el-row :gutter="10">
    <el-col :span="12"  style="height: 100%;" class='left-box'>
        <div class='info-item stu-name'>{{stuinfo.name}}</div>
        <div class='info-item'><span class='item-name'>学号</span><span class='item-detail'>{{stuinfo.id}}</span></div>
        <div class='info-item'><span class='item-name'>班级</span><span class='item-detail'>{{stuinfo.class_id}}</span></div>
        <div class='info-item'><span class='item-name'>性别</span><span class='item-detail'>{{stuinfo.sex == '1'?'男': '女'}}</span></div>
        <div class='info-item'><span class='item-name'>寝室号</span><span class='item-detail'>{{stuinfo.room_num == null?'不住校': stuinfo.room_num}}</span></div>

    </el-col>
    <el-col :span="12" style="height: 100%;" class='right-box'>
        <div class='info-item'><span class='item-name'>出生日期</span><span class='item-detail'>{{stuinfo.born_year}}</span></div>
        <div class='info-item'><span class='item-name'>政治面貌</span><span class='item-detail'>{{stuinfo.policy}}</span></div>
        <div class='info-item'><span class='item-name'>民族</span><span class='item-detail'>{{stuinfo.nation}}</span></div>
        <div class='info-item'><span class='item-name'>户籍</span><span class='item-detail'>{{stuinfo.native_place}}</span></div>
        <div class='info-item'><span class='item-name'>户籍类型</span><span class='item-detail'>{{stuinfo.residence_type}}</span></div>
    </el-col>

  </el-row>
</template>

<script>
import axios from 'axios'

  export default {
    props: ['student_id'],
    data() {
      return {
        stuinfo: {}
      }
    },
    methods: {
      getData(){
        const request = axios.create({
          baseURL: 'http://localhost:3000'
        });

        request({
          url: '/stu/selectStu',
          params: {
            student_id: this.student_id
          }
        }).then(res => {
          this.stuinfo = res.data[0]
        })
      }
    },
    created(){
      this.getData();
    },
    watch: {
        student_id: function(){
          this.getData();
        }
    }
  }
</script>

<style >
    .left-box {
        border-right: 1px solid rgb(233, 233, 233);
    }

    .stu-name {
       text-align: center;
       font-weight: bolder;
    }

    .info-item {
        height: 28px;
        line-height: 28px;
        padding: 0 15px;
    }
    .info-item .item-name{
      font-size: 12px;
      color: rgb(98, 111, 126);
    }
    .info-item .item-detail{
      font-size: 10px;
      color: rgb(158, 158, 158);
      float: right;
    }
</style>