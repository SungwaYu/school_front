<template>

  <!-- 添加和修改章节表单 -->
  <el-dialog :visible="dialogVisible" title="Add chapter" @close="close()">
    <el-form :model="chapter" label-width="120px">
      <el-form-item label="Chapter title">
        <el-input v-model="chapter.title"/>
      </el-form-item>
      <el-form-item label="Chapter order">
        <el-input-number v-model="chapter.sort" :min="0"/>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="close()">Cancel</el-button>
      <el-button type="primary" @click="saveOrUpdate()">Confirm</el-button>
    </div>
  </el-dialog>

</template>

<script>
import chapterApi from '@/api/chapter'

export default {

  data() {
    return {
      dialogVisible: false, // 是否显示对话框
      chapter: { // 章节对象
        sort: 0
      }
    }
  },

  methods: {

    // 显示对话框
    open(chapterId) {
      this.dialogVisible = true
      if (chapterId) {
        chapterApi.getById(chapterId).then(response => {
          this.chapter = response.data.item
        })
      }
    },

    // 关闭对话框
    close() {
      this.dialogVisible = false
      this.resetForm()
    },

    // 重置表单
    resetForm() {
      this.chapter = {
        sort: 0
      }
    },

    // 保存或更新章节
    saveOrUpdate() {
      if (!this.chapter.id) {
        this.save()
      } else {
        this.update()
      }
    },

    // 保存章节
    save() {
      this.chapter.courseId = this.$parent.$parent.courseId
      chapterApi.save(this.chapter).then(response => {
        this.$message.success(response.message)
        // 关闭组件
        this.close()
        // 刷新列表
        this.$parent.fetchNodeList()
      })
    },

    // 更新章节
    update() {
      chapterApi.updateById(this.chapter).then(response => {
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
