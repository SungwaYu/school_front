
<template>
  <div class="app-container">
    <el-form label-width="120px">

      <el-form-item label="Info description">

        <el-tag type="info">excel template</el-tag>
        <el-tag>
          <i class="el-icon-download"/>
          <a :href="defaultExcelTemplate">Download template</a>
        </el-tag>

      </el-form-item>

      <el-form-item label="Select Excel">
        <el-upload
          ref="upload"
          :auto-upload="false"
          :on-exceed="fileUploadExceed"
          :on-success="fileUploadSuccess"
          :on-error="fileUploadError"
          :limit="1"
          action="http://127.0.0.1:8110/admin/edu/subject/import"
          name="file"
          accept="application/vnd.ms-excel">
          <el-button
            slot="trigger"
            size="small"
            type="primary">Select file</el-button>
          <el-button
            :disabled="importBtnDisabled"
            style="margin-left: 10px;"
            size="small"
            type="success"
            @click="submitUpload()">Import</el-button>
        </el-upload>

      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      defaultExcelTemplate: process.env.OSS_PATH + '/excel/课程分类列表模板.xls', // 默认Excel模板
      importBtnDisabled: false // 导入按钮是否禁用
    }
  },

  methods: {

    // 执行上传
    submitUpload() {
      this.importBtnDisabled = true // 禁用按钮
      this.$refs.upload.submit() // 手动表单提交
    },

    // 当选择文件超出约定数量时触发
    fileUploadExceed() {
      this.$message.warning('只能选取一个文件')
    },

    // 上传成功的回调
    fileUploadSuccess(response) {
      if (response.success) {
        this.importBtnDisabled = false // 启用按钮
        this.$message.success(response.message)
        this.$refs.upload.clearFiles() // 清空文件列表
      } else {
        this.$message.error('上传失败! （非20000）')
      }
    },

    // 上传失败的回调
    fileUploadError() {
      this.importBtnDisabled = false // 启用按钮
      this.$message.error('上传失败! （http失败）')
      this.$refs.upload.clearFiles() // 清空文件列表
    }
  }
}
</script>
