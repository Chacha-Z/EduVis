<template>
  <div id='topnavi'>
    <el-select class='class-select' v-model="value" style="width: 10em; "  @change="$emit('class-change', value)">
        <el-option
        v-for="item in options"
        :key="item.value"
        :label="item.label"
        :value="item.value">
        </el-option>
    </el-select>
  </div>
</template>

<script>
import axios from 'axios'
  export default {
    props: ['class_id'],
    data() {
      return {
        options: [],
        value: this.class_id
      }
    },
    methods: {
      getData(){
        const request = axios.create({
          baseURL: 'http://localhost:3000'
        });

        request({
          url: '/stu/selectCla',
        }).then(res => {
          this.options = res.data;
          this.value = this.options[0].value
          this.$emit('class-change', this.value)
        })

      }
    },
    created(){
      this.getData();
    },
  }
</script>

<style>

    .class-select .el-input__inner {
        border: none;
        font-weight: bolder;
    }

    .el-select-dropdown .el-select-dropdown__item {
        font-size: 10px;
        padding: 0;
        height: 25px;
        line-height: 25px;
        text-align: center;
    }

    div#topnavi {
      width: 100%;
      background-color: #fff;
    }

</style>