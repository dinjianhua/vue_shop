<template>
  <el-container class="home-container">
    <!-- 头部 -->
    <el-header>
      <div class="">
        <img src="../assets/images/heima.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px': '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单区 -->
        <el-menu
          :unique-opened="true"
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409bff">
          <!-- 一级菜单 -->
          <el-submenu :index="String(item.id)" v-for="item in menulist" :key="item.id">
            <!-- 一级菜单模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item @click="saveNavState('/' + subItem.path)" :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id">
              <template slot="title">
              <!-- 图标 -->
              <i class="el-icon-menu"></i>
              <!-- 文本 -->
              <span>{{subItem.authName}}</span>
            </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 内容主体 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      // 左侧菜单数据
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  created () {
    this.getData()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.removeItem('token')
      this.$router.push('/login')
    },
    async getData () {
      const { data: res } = await this.$axios.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    // 点击按钮，切换按钮的折叠与展开
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  background-color: #373d41;
  color: #fff;
  font-size: 20px;
  >div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  transition: width .1s ease-in-out;
  background-color: #333744;
}
.el-main {
  background-color: #eaedf1;
}
.el-menu {
  border: none;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
