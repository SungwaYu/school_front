<template>
  <!-- 公共头 -->
  <header id="header">
    <section class="container">
      <h1 id="logo">
        <a href="#" title="SchoolProject">
          <img src="~/assets/img/logo1.png" width="100%" alt="School Logo">
        </a>
      </h1>
      <div class="h-r-nsl">
        <ul class="nav">
          <router-link to="/" tag="li" active-class="current" exact>
            <a>Home</a>
          </router-link>
          <router-link to="/course" tag="li" active-class="current">
            <a>Sourse</a>
          </router-link>
          <router-link to="/teacher" tag="li" active-class="current">
            <a>Teacher</a>
          </router-link>
          <router-link to="/article" tag="li" active-class="current">
            <a>Article</a>
          </router-link>
          <router-link to="/question" tag="li" active-class="current">
            <a>Q&A</a>
          </router-link>
        </ul>
        <!-- / nav -->
        <ul class="h-r-login">
          <li v-if="!userInfo" id="no-login">
            <a href="/login" title="Log in">
              <em class="icon18 login-icon">&nbsp;</em>
              <span class="vam ml5">Log in</span>
            </a>
            |
            <a href="/register" title="Sign up">
              <span class="vam ml5">Sign up</span>
            </a>
          </li>
          <!-- 注意undis将当前节点隐藏了 -->
          <li v-if="userInfo" id="is-login-one" class="mr10">
            <a id="headerMsgCountId" href="#" title="Message">
              <em class="icon18 news-icon">&nbsp;</em>
            </a>
            <q class="red-point">&nbsp;</q>
          </li>
          <!-- 注意undis将当前节点隐藏了 -->
          <li v-if="userInfo" id="is-login-two" class="h-r-user">
            <a href="/ucenter" title>
              <img
                :src="userInfo.avatar"
                width="30"
                height="30"
                class="vam picImg"
                alt
              >
              <span id="userName" class="vam disIb">{{ userInfo.nickname }}</span>
            </a>
            <a href="javascript:void(0)" title="Log out" class="ml5" @click="logout()">Log out</a>
          </li>
          <!-- /未登录显示第1 li；登录后显示第2，3 li -->
        </ul>
        <aside class="h-r-search">
          <form action="#" method="post">
            <label class="h-r-s-box">
              <input type="text" placeholder="Search Classes" name="queryCourse.courseName" value>
              <button type="submit" class="s-btn">
                <em class="icon18">&nbsp;</em>
              </button>
            </label>
          </form>
        </aside>
      </div>
      <aside class="mw-nav-btn">
        <div class="mw-nav-icon"/>
      </aside>
      <div class="clear"/>
    </section>
  </header>
  <!-- /公共头 -->
</template>

<script>
import loginApi from '~/api/login'
import cookie from 'js-cookie'

export default {
  data() {
    return {
      userInfo: null
    }
  },

  // 登录成功后获取用户信息
  // 页面渲染之前执行：尚未有window对象
  created() {
    this.getUserInfo()
  },

  // 页面渲染之后执行：可以有window对象
  mounted() {
    const token = this.$route.query.token
    if (token) {
      // 将jwt写入cookie
      cookie.set('guli_jwt_token', token, { domain: 'localhost' })
      window.location.href = '/'
    }
  },

  methods: {
    getUserInfo() {
      // 如果cookie中的token值不存在，则无需获取用户信息
      if (!cookie.get('guli_jwt_token')) {
        return
      }

      // 如果token村子，则携带token的值向服务器发起请求
      loginApi.getLoginInfo().then(response => {
        // 渲染页面
        this.userInfo = response.data.userInfo
      })
    },

    logout() {
      // 清除cookie
      cookie.set('guli_jwt_token', '', { domain: 'localhost' })
      // 跳转页面
      window.location.href = '/'
    }
  }
}
</script>
