<template>
  <el-select class='my-select' v-model="value" style="width: 16em"  @change="$emit('exam-change', value)">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
  </el-select>
</template>

<script>
import axios from 'axios'
  export default {
    props: ['student_id', 'exam_value'],
    data() {
      return {
        options: [],
        value: ''
      }
    },
    methods: {
      getData(){
        const request = axios.create({
          baseURL: 'http://localhost:3000'
        });

        request({
          url: '/stu/selectExamID',
          params: {
            student_id: this.student_id,
          }
        }).then(res => {
          this.options = res.data;
          this.value = this.options[0].value
          this.$emit('exam-change', this.value)
        })

      }
    },
    created(){
      this.getData();
    },
    watch: { 
        student_id: function(){
            // console.log('exambox watch student_id: ', this.student_id);
            this.getData()
        },
        exam_value: function(){
          this.value = this.exam_value;
        }
    }
  }
</script>

<style>
    .my-select .el-input {
        font-size: 10px;
    }

    .my-select .el-input__inner {
        height: 30px;
        line-height: 30px;
        border: none;
    }

    .my-select .el-input__icon {
        line-height: 30px;
    }
    .el-select-dropdown .el-select-dropdown__item {
        font-size: 10px;
        padding: 0;
        height: 25px;
        line-height: 25px;
        text-align: center;
    }


</style>