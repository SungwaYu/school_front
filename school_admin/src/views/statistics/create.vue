<template>
  <div class="app-container">
    <!--表单-->
    <el-form :inline="true" class="demo-form-inline">

      <el-form-item label="Date">
        <el-date-picker
          v-model="day"
          type="date"
          placeholder="Please select a date"
          value-format="yyyy-MM-dd" />
      </el-form-item>

      <el-button
        :disabled="btnDisabled"
        type="primary"
        @click="genarateData()">Generate</el-button>
    </el-form>

  </div>
</template>

<script>
import statisticsApi from '@/api/statistics'

export default {
  data() {
    return {
      day: '',
      btnDisabled: false // 按钮禁用状态
    }
  },

  methods: {

    // 生成统计数据
    genarateData() {
      this.btnDisabled = true
      statisticsApi.createStatistics(this.day).then(response => {
        this.btnDisabled = false
        this.$message.success(response.message)
      })
    }
  }
}
</script>
