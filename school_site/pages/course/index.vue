<template>
  <div id="aCoursesList" class="bg-fa of">
    <!-- 课程列表 开始 -->
    <section class="container">
      <header class="comm-title">
        <h2 class="fl tac">
          <span class="c-333">All courses</span>
        </h2>
      </header>
      <section class="c-sort-box">
        <section class="c-s-dl">
          <dl>
            <dt>
              <span class="c-999 fsize14">Type</span>
            </dt>
            <dd class="c-s-dl-li">
              <!-- 一级类别 开始-->
              <ul class="clearfix">
                <li :class="{current:!$route.query.subjectParentId}">
                  <a
                    title="All"
                    href="javascript:void(0);"
                    @click="searchSubjectLevelOne('')">All</a>
                </li>
                <li
                  v-for="item in subjectNestedList"
                  :key="item.id"
                  :class="{current:$route.query.subjectParentId===item.id}">
                  <a
                    :title="item.title"
                    href="javascript:void(0);"
                    @click="searchSubjectLevelOne(item.id)">{{ item.title }}</a>
                </li>
              </ul>
              <!-- /一级类别 结束-->
            </dd>
          </dl>
          <dl>
            <dt>
              <span class="c-999 fsize14"/>
            </dt>
            <dd class="c-s-dl-li">
              <!-- 二级类别 开始-->
              <ul v-if="$route.query.subjectParentId" class="clearfix">
                <li :class="{current:!$route.query.subjectId}">
                  <a
                    title="All"
                    href="javascript:void(0);"
                    @click="searchSubjectLevelTwo('')">All</a>
                </li>
                <li
                  v-for="item in subSubjectList"
                  :key="item.id"
                  :class="{current:$route.query.subjectId===item.id}">
                  <a
                    :title="item.title"
                    href="javascript:void(0);"
                    @click="searchSubjectLevelTwo(item.id)">{{ item.title }}</a>
                </li>
              </ul>
              <!-- /二级类别 结束-->
            </dd>
          </dl>
          <div class="clear"/>
        </section>
        <div class="js-wrap">
          <section class="fr">
            <span class="c-ccc">
              <i class="c-master f-fM">1</i>/
              <i class="c-666 f-fM">1</i>
            </span>
          </section>
          <section class="fl">
            <!-- 排序 开始-->
            <ol class="js-tap clearfix">
              <li :class="{'current bg-green': $route.query.buyCountSort}">
                <a title="Sold" href="javascript:void(0);" @click="searchBuyCount()">Sold
                  <i>↓</i>
                </a>
              </li>
              <li :class="{'current bg-green': $route.query.gmtCreateSort}">
                <a title="Lastest" href="javascript:void(0);" @click="searchGmtCreate()">Latest
                  <i>↓</i>
                </a>
              </li>
              <li :class="{'current bg-green': $route.query.priceSort}">
                <a v-if="!$route.query.type || $route.query.type == 1" title="Price" href="javascript:void(0);" @click="searchPrice(2)">Price
                  <i>↑</i>
                </a>

                <a v-if="$route.query.type == 2" title="Price" href="javascript:void(0);" @click="searchPrice(1)">Price
                  <i>↓</i>
                </a>
              </li>
            </ol>
            <!-- /排序 结束-->
          </section>
        </div>
        <div class="mt40">
          <!-- /无数据提示 开始-->
          <section v-if="courseList.length===0" class="no-data-wrap">
            <em class="icon30 no-data-ico">&nbsp;</em>
            <span class="c-666 fsize14 ml10 vam">No course is upload yet</span>
          </section>
          <!-- /无数据提示 结束-->

          <!-- 数据列表 开始-->
          <article v-if="courseList.length>0" class="comm-course-list">
            <ul id="bna" class="of">
              <li v-for="item in courseList" :key="item.id">
                <div class="cc-l-wrap">
                  <section class="course-img">
                    <img :src="item.cover" :alt="item.title" class="img-responsive">
                    <div class="cc-mask">
                      <a :href="'/course/'+item.id" title="Start" class="comm-btn c-btn-1">Start</a>
                    </div>
                  </section>
                  <h3 class="hLh30 txtOf mt10">
                    <a :href="'/course/'+item.id" :title="item.title" class="course-title fsize18 c-333">{{ item.title }}</a>
                  </h3>
                  <section class="mt10 hLh20 of">
                    <span v-if="Number(item.price) === 0" class="fr jgTag bg-green">
                      <i class="c-fff fsize12 f-fA">Free</i>
                    </span>
                    <span v-else class="fr jgTag ">
                      <i class="c-orange fsize12 f-fA"> ${{ item.price }}</i>
                    </span>
                    <span class="fl jgAttr c-ccc f-fA">
                      <i class="c-999 f-fA">{{ item.viewCount }}learning</i>
                      |
                      <i class="c-999 f-fA">{{ item.buyCount }}sold</i>
                    </span>
                  </section>
                </div>
              </li>
            </ul>
            <div class="clear"/>
          </article>
          <!-- /数据列表 结束-->
        </div>
      </section>
    </section>
    <!-- /课程列表 结束 -->
  </div>
</template>

<script>
import courseApi from '~/api/course'
import querystring from 'querystring' // url参数拼接工具

export default {

  async asyncData(page) {
    // 组装查询参数
    // 当点击查询条件的时候：在url地址栏中组装查询参数
    // 从url地址栏中获取查询参数，对页面中响应部分的内容进行高亮显示
    const searchObj = {}
    const query = page.route.query
    searchObj.subjectParentId = query.subjectParentId || ''
    searchObj.subjectId = query.subjectId || ''
    searchObj.buyCountSort = query.buyCountSort || ''
    searchObj.gmtCreateSort = query.gmtCreateSort || ''
    searchObj.priceSort = query.priceSort || ''
    searchObj.type = query.type || '' // 1：正序，2：倒序

    // 课程列表的查询
    const courseListResponse = await courseApi.getList(searchObj)
    const courseList = courseListResponse.data.courseList
    // console.log('courseList', courseList)

    // 课程分类列表的查询
    const subjectNestedListResponse = await courseApi.getSubjectNestedList()
    const subjectNestedList = subjectNestedListResponse.data.items
    // 遍历一级分类：查询一级分类下的二级分类
    let subSubjectList = []
    for (let i = 0; i < subjectNestedList.length; i++) {
      if (subjectNestedList[i].id === searchObj.subjectParentId) {
        // 组装当前一级分类下的二级分类数据
        subSubjectList = subjectNestedList[i].children
      }
    }

    return {
      courseList, // 课程列表
      subjectNestedList, // 课程分类列表
      subSubjectList, // 课程二级分类列表
      searchObj // 查询对象
    }
  },

  methods: {

    // 选择一级分类
    searchSubjectLevelOne(subjectParentId) {
      // 让页面根据新的url地址刷新
      window.location = 'course?subjectParentId=' + subjectParentId
    },

    // 选择二级分类
    searchSubjectLevelTwo(subjectId) {
      // console.log(this.searchObj)
      // window.location = 'course?subjectId=' + subjectId + '&subjectParentId=' + this.searchObj.subjectParentId
      const queryObj = {
        subjectId: subjectId,
        subjectParentId: this.searchObj.subjectParentId
      }
      const querys = querystring.stringify(queryObj)
      // console.log(querys)
      window.location = 'course?' + querys
    },

    // 选择按销量倒序
    searchBuyCount() {
      // console.log(this.searchObj)
      // window.location = 'course?buyCountSort=1' + '&subjectId=' + this.searchObj.subjectId + '&subjectParentId=' + this.searchObj.subjectParentId

      const queryObj = {
        subjectId: this.searchObj.subjectId,
        subjectParentId: this.searchObj.subjectParentId,
        buyCountSort: 1
      }
      const querys = querystring.stringify(queryObj)
      // console.log(querys)
      window.location = 'course?' + querys
    },

    // 按创建时间排序
    searchGmtCreate() {
      const queryObj = {
        subjectId: this.searchObj.subjectId,
        subjectParentId: this.searchObj.subjectParentId,
        gmtCreateSort: 1
      }
      const querys = querystring.stringify(queryObj)
      // console.log(querys)
      window.location = 'course?' + querys
    },

    // 按价格排序
    searchPrice(type) {
      const queryObj = {
        subjectId: this.searchObj.subjectId,
        subjectParentId: this.searchObj.subjectParentId,
        priceSort: 1,
        type: type
      }
      const querys = querystring.stringify(queryObj)
      // console.log(querys)
      window.location = 'course?' + querys
    }
  }
}
</script>
