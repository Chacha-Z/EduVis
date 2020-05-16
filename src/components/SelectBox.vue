<template>
  <el-select class='my-select' v-model="value" placeholder="科目" style="width: 6em"  @change="$emit('select-change', value)">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
  </el-select>
</template>

<script>
  export default {
    data() {
      return {
        options: [{
          value: '0',
          label: '总分'
        },{
          value: '1',
          label: '语文'
        }, {
          value: '2',
          label: '数学'
        }, {
          value: '3',
          label: '外语'
        }, {
          value: '4',
          label: '物理'
        }, {
          value: '5',
          label: '化学'
        }, {
          value: '6',
          label: '生物'
        }, {
          value: '7',
          label: '历史'
        }, {
          value: '8',
          label: '地理'
        }, {
          value: '17',
          label: '政治'
        }],
        value: '0'
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