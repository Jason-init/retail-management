<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="logo">
        <span>电商管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 左侧导航栏 -->
      <el-aside :width="isCollapsed ? '64px':'200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409BFF" unique-opened :collapse="isCollapsed" :collapse-transition="false" :router="true" :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="menuIcons[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 一级菜单选项 -->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容区域 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data: function () {
    return {
      menuList: [],
      menuIcons: {
        125: 'iconfont icon-yonghu',
        103: 'iconfont icon-ic_opt_feature',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-dingdan',
        145: 'iconfont icon-baobiao'
      },
      isCollapsed: false,
      activePath: ''
    }
  },
  created: function () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout: function () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    getMenuList: async function () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    toggleCollapse: function () {
      this.isCollapsed = !this.isCollapsed
    },
    saveNavState: function (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
    }
  }
}
</script>

<style lang="less" scoped>.el-header {
  background-color: #373D41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  color: #fff;
  font-size: 20px;
  align-items: center;

  >div {
    display: flex;
    align-items: center;

    >span {
      margin-left: 15px;
    }
  }
}

.el-aside {
  background-color: #333744;

  >.el-menu {
    border-right: 0;
  }
}

.el-main {
  background-color: #EAEDF1;
}

.home-container {
  height: 100%;
}

.iconfont {
  margin-right: 10px;
}

.toggle-button {
  background-color: #4A5064;
  cursor: pointer;
  color: #fff;
  text-align: center;
  font-size: 10px;
  line-height: 25px;
  letter-spacing: 0.2em;
}

</style>
