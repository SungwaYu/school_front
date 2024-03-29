<template>
  <!-- 添加和修改课时表单 -->
  <el-dialog :visible="dialogVisible" title="Add unit" @close="close()">
    <el-form :model="video" label-width="120px">
      <el-form-item label="Unit title">
        <el-input v-model="video.title"/>
      </el-form-item>
      <el-form-item label="Unit order">
        <el-input-number v-model="video.sort" :min="0" />
      </el-form-item>
      <el-form-item label="Free?">
        <el-radio-group v-model="video.free">
          <el-radio :label="true">Free</el-radio>
          <el-radio :label="false">Default</el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- 上传视频 -->
      <el-form-item label="Upload video">
        <el-upload
          ref="upload"
          :auto-upload="false"
          :on-success="handleUploadSuccess"
          :on-error="handleUploadError"
          :on-exceed="handleUploadExceed"
          :before-remove="handleBeforeRemove"
          :on-remove="handleOnRemove"
          :file-list="fileList"
          :limit="1"
          :action="BASE_API+'/admin/vod/media/upload'">
          <el-button slot="trigger" size="small" type="primary">Select video</el-button>
          <el-button
            :disabled="uploadBtnDisabled"
            style="margin-left: 10px;"
            size="small"
            type="success"
            @click="submitUpload()">Upload</el-button>
        </el-upload>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="close()">Cancel</el-button>
      <el-button type="primary" @click="saveOrUpdate()">Confirm</el-button>
    </div>
  </el-dialog>
</template>

<script>
import videoApi from '@/api/video'
import vodApi from '@/api/vod'

export default {
  data() {
    return {
      dialogVisible: false, // 是否显示对话框
      video: {
        sort: 0,
        free: false
      },
      fileList: [], // 上传文件列表
      uploadBtnDisabled: false,
      BASE_API: process.env.BASE_API
    }
  },

  methods: {

    // 上传多于一个视频
    handleUploadExceed(files, fileList) {
      this.$message.warning('想要重新上传视频，请先删除已上传的视频')
    },

    // 上传
    submitUpload() {
      this.uploadBtnDisabled = true// 禁用按钮
      this.$refs.upload.submit() // 提交上传请求
    },

    // 视频上传成功的回调
    handleUploadSuccess(response, file, fileList) {
      this.uploadBtnDisabled = false
      if (response.success) {
        this.video.videoSourceId = response.data.videoId
        this.video.videoOriginalName = file.name

        // debugger
      } else {
        this.$message.error('上传失败（非20000）')
      }
    },

    // 失败回调
    handleUploadError() {
      this.uploadBtnDisabled = false
      this.$message.error('上传失败（http）')
    },

    // 删除视频文件确认
    handleBeforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },

    // 删除视频
    handleOnRemove(file, fileList) {
      if (!this.video.videoSourceId) {
        return
      }

      vodApi.removeByVodId(this.video.videoSourceId).then(response => {
        this.video.videoSourceId = ''
        this.video.videoOriginalName = ''
        // 删除视频的同时更新video数据信息
        videoApi.updateById(this.video)
        this.$message.success(response.message)
      })
    },

    // 显示对话框
    open(chapterId, videoId) {
      this.dialogVisible = true
      this.video.chapterId = chapterId
      if (videoId) {
        videoApi.getById(videoId).then(response => {
          this.video = response.data.item
          // 回显
          // debugger
          if (this.video.videoSourceId) {
            this.fileList = [{ 'name': this.video.videoOriginalName }]
          }
        })
      }
    },

    // 关闭页面
    close() {
      this.dialogVisible = false
      // 重置表单
      this.resetForm()
    },

    resetForm() {
      this.video = {
        sort: 0,
        free: false
      }

      // 清空文件列表
      this.fileList = []
    },

    // 保存和更新
    saveOrUpdate() {
      if (!this.video.id) {
        this.save()
      } else {
        this.update()
      }
    },

    save() {
      this.video.courseId = this.$parent.$parent.courseId
      videoApi.save(this.video).then(response => {
        this.$message.success(response.message)
        // 关闭组件
        this.close()
        // 刷新列表
        this.$parent.fetchNodeList()
      })
    },

    update() {
      // debugger
      videoApi.updateById(this.video).then(response => {
        // debugger
        this.$message.success(response.message)
        // 关闭组件
        this.close()
        // 刷新列表
        this.$parent.fetchNodeList()
      })
    }
  }
}
</script>
