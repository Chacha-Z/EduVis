<template>
    <div>
      <el-switch
        v-model="value"
        active-text="隐藏平时成绩"
        @change="$emit('conceal-change', value)">
      </el-switch>
    </div>
</template>


<script>
  export default {
    props:['conceal_usual_performance'],
    data() {
      return {
        value: this.conceal_usual_performance,
      }
    }
  };
</script>

<style>
  .el-switch .el-switch__core {
    height: 15px;
  }
  .el-switch .el-switch__label *{
    font-size: 10px;
  }
  .el-switch .el-switch__core:after{
    width: 14px;
    height: 14px;
    top: 0;
    left: 0;
  }
  div .el-switch.is-checked .el-switch__core::after{
    margin-left: -14px;
  }
</style>