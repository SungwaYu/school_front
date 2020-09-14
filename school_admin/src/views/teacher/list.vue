<template>
  <div class="app-container">
    <!--查询表单-->
    <el-form :inline="true">
      <el-form-item>
        <!-- <el-input v-model="searchObj.name" placeholder="讲师"/> -->
        <el-autocomplete
          v-model="searchObj.name"
          :fetch-suggestions="querySearch"
          :trigger-on-focus="false"
          class="inline-input"
          placeholder="teacher"
          value-key="name" />
      </el-form-item>

      <el-form-item>
        <el-select v-model="searchObj.level" clearable placeholder="Level">
          <el-option value="1" label="Experience"/>
          <el-option value="2" label="Expert"/>
        </el-select>
      </el-form-item>

      <el-form-item label="Join Time">
        <el-date-picker
          v-model="searchObj.joinDateBegin"
          placeholder="Starting Time"
          value-format="yyyy-MM-dd" />
      </el-form-item>
      <el-form-item label="-">
        <el-date-picker
          v-model="searchObj.joinDateEnd"
          placeholder="Ending Time"
          value-format="yyyy-MM-dd" />
      </el-form-item>

      <el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="fetchData()">Search</el-button>
        <el-button type="default" @click="resetData()">Clear</el-button>
      </el-form-item>
    </el-form>

    <!-- 工具条 -->
    <div style="margin-bottom: 10px;">
      <el-button type="danger" size="mini" @click="batchRemove()">Delete selected</el-button>
    </div>

    <!-- 表格 -->
    <el-table
      :data="list"
      border
      stripe
      @selection-change="handleSelectionChange">

      <el-table-column width="40" type="selection" />

      <el-table-column
        label="#"
        width="30">
        <template slot-scope="scope">
          {{ (page - 1) * limit + scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column prop="name" label="Name" width="150" />
      <el-table-column label="Level" width="100">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.level === 1" type="success" size="mini">Experiencer</el-tag>
          <el-tag v-if="scope.row.level === 2" size="mini">Expert</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="intro" label="Intro" />
      <el-table-column prop="sort" label="Sort" width="50" />
      <el-table-column prop="joinDate" label="Join Time" width="100" />
      <el-table-column label="Operation" width="200">
        <template slot-scope="scope">
          <router-link :to="'/teacher/edit/'+scope.row.id">
            <el-button type="primary" size="mini" icon="el-icon-edit">Update</el-button>
          </router-link>
          <el-button
            size="mini"
            type="danger"
            @click="removeById(scope.row.id)">Delete</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      :current-page="page"
      :total="total"
      :page-size="limit"
      :page-sizes="[5, 10, 20]"
      style="padding: 30px 0; text-align: center"
      layout="sizes, prev, pager, next, jumper, ->, total"
      @current-change="changeCurrentPage"
      @size-change="changePageSize"/>
  </div>
</template>

<script>
import teacherApi from '@/api/teacher'

export default {

  data() {
    return {
      list: [],
      total: 0,
      page: 1,
      limit: 5,
      searchObj: {},
      multipleSelection: [] // 批量选中的记录列表
    }
  },

  created() {
    this.fetchData()
  },

  methods: {
    fetchData() {
      teacherApi.pageList(this.page, this.limit, this.searchObj).then(response => {
        this.list = response.data.rows
        this.total = response.data.total
      })
    },

    changeCurrentPage(page) {
      console.log('changeCurrentPage:' + page)
      this.page = page
      this.fetchData()
    },

    // 改变每页记录数
    changePageSize(size) {
      console.log('changePageSize:' + size)
      this.limit = size
      this.fetchData()
    },

    // 重置表单
    resetData() {
      this.searchObj = {}
      this.fetchData()
    },

    // 当表格中多选项发生变化的时候触发
    handleSelectionChange(selection) {
      this.multipleSelection = selection
    },

    // 批量删除
    batchRemove() {
      // 判断是否选中记录
      if (this.multipleSelection.length === 0) {
        this.$message({
          type: 'warning',
          message: 'Please select before operating'
        })
        return
      }
      // 询问是否删除
      this.$confirm('Data will be remove permenantly, are you sure?', '提示', {
        confirmButtonText: 'Yes',
        cancelButtonText: 'No',
        type: 'warning'
      }).then(() => {
        // 拿到选中数据
        const idList = []
        this.multipleSelection.forEach(item => {
          idList.push(item.id)
        })
        return teacherApi.batchRemove(idList)
      }).then(response => {
        // 刷新页面
        this.fetchData()
        // 弹出成功提示
        this.$message({
          message: response.message,
          type: 'success'
        })
      }).catch((err) => {
        console.log(err)
        if (err === 'cancel') {
          this.$message({
            type: 'info',
            message: 'Suceess Deleted'
          })
        }
      })
    },
    // 删除记录
    removeById(id) {
      // 询问是否删除
      this.$confirm('Data will be remove permenantly, are you sure??', '提示', {
        confirmButtonText: 'Yes',
        cancelButtonText: 'No',
        type: 'warning'
      }).then(() => {
        return teacherApi.removeById(id)
      }).then(response => {
        // 刷新页面
        this.fetchData()
        // 弹出成功提示
        this.$message({
          message: response.message,
          type: 'success'
        })
      }).catch((err) => {
        console.log(err)
        if (err === 'cancel') {
          this.$message({
            type: 'info',
            message: 'Suceess Deleted'
          })
        }
      })
    },

    // 输入建议
    querySearch(queryString, callback) {
      teacherApi.selectNameListByKey(queryString).then(response => {
        callback(response.data.nameList)
      })
    }
  }
}
</script>
