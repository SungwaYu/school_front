<template>
  <div class="app-container">
    <!-- 输入表单 -->
    <el-form label-width="140px">
      <el-form-item label="Ads name">
        <el-input v-model="ad.title" />
      </el-form-item>
      <!-- 推荐位 -->
      <el-form-item label="Recommend">
        <el-select
          v-model="ad.typeId"
          placeholder="Please select">
          <el-option
            v-for="adType in adTypeList"
            :key="adType.id"
            :label="adType.title"
            :value="adType.id"/>
        </el-select>
      </el-form-item>
      <el-form-item label="Sort">
        <el-input-number v-model="ad.sort" :min="0"/>
      </el-form-item>
      <el-form-item label="Ad imange">
        <el-upload
          :on-success="handleAvatarSuccess"
          :on-error="handleAvatarError"
          :on-exceed="handleUploadExceed"
          :before-upload="beforeAvatarUpload"
          :limit="1"
          :file-list="fileList"
          :action="BASE_API+'/admin/oss/file/upload?module=ad'"
          list-type="picture">
          <el-button size="small" type="primary">Upload</el-button>
        </el-upload>
      </el-form-item>
      <el-form-item label="Background color">
        <el-color-picker v-model="ad.color"/>
      </el-form-item>
      <el-form-item label="Link">
        <el-input v-model="ad.linkUrl" />
      </el-form-item>
      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate()">Save</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import adApi from '@/api/ad'
import adTypeApi from '@/api/adType'

export default {
  data() {
    return {
      ad: {
        sort: 0
      },
      fileList: [], // 上传文件列表
      adTypeList: [],
      saveBtnDisabled: false, // 保存按钮是否禁用，防止表单重复提交
      BASE_API: process.env.BASE_API
    }
  },

  // 页面渲染成功
  created() {
    if (this.$route.params.id) {
      this.fetchDataById(this.$route.params.id)
    }

    // 获取推荐位列表
    this.initAdTypeList()
  },

  methods: {

    initAdTypeList() {
      adTypeApi.list().then(response => {
        this.adTypeList = response.data.items
      })
    },

    saveOrUpdate() {
      // 禁用保存按钮
      this.saveBtnDisabled = true
      if (!this.ad.id) {
        this.saveData()
      } else {
        this.updateData()
      }
    },

    // 新增
    saveData() {
      // debugger
      adApi.save(this.ad).then(response => {
        this.$message.success(response.message)
        this.$router.push({ path: '/ad/list' })
      })
    },

    // 根据id更新记录
    updateData() {
      // teacher数据的获取
      adApi.updateById(this.ad).then(response => {
        this.$message.success(response.message)
        this.$router.push({ path: '/ad/list' })
      })
    },

    // 根据id查询记录
    fetchDataById(id) {
      adApi.getById(id).then(response => {
        this.ad = response.data.item
        this.fileList = [{ 'url': this.ad.imageUrl }]
      })
    },

    // 上传多于一个文件
    handleUploadExceed(files, fileList) {
      this.$message.warning('Please delte the previous video before upload the image')
    },

    // 上传校验
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('Image must be JPG format')
      }
      if (!isLt2M) {
        this.$message.error('Image cannot more than 2MB')
      }
      return isJPG && isLt2M
    },

    // 上传成功回调
    handleAvatarSuccess(res, file) {
      console.log(res)
      if (res.success) {
        // console.log(res)
        this.ad.imageUrl = res.data.url
        // 强制重新渲染
        // this.$forceUpdate()
      } else {
        this.$message.error('Upload failed1')
      }
    },

    // 错误处理
    handleAvatarError() {
      console.log('error')
      this.$message.error('Upload failed2')
    }
  }
}
</script>
