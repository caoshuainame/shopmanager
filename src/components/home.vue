<template>
    <el-container class="container">
      <!-- 顶部导航栏 -->
      <el-header>
            <el-row>
                <el-col :span="4">
                    <img src="@/assets/logo.png" alt="">
                </el-col>
                <el-col :span="19" class="middle">
                    <h2>电商后台管理系统</h2>
                </el-col>
                <el-col :span="1">
                    <div class="grid-content bg-purple">
                        <a href="#" @click.prevent="handleLoginout()" class="loginout">退出</a>
                    </div>
                </el-col>
            </el-row>
      </el-header>
      <el-container>
      <!-- 侧边栏 -->
        <el-aside class="aside" width="200px">
            <el-menu default-active="1" :router="true" :unique-opened="true">
                <!-- 用户管理 -->
                <el-submenu :index="item1.order+''" v-for="(item1) in menus" :key="item1.id">
                  <template slot="title">
                    <i class="el-icon-location"></i>
                    <span>{{item1.authName}}</span>
                  </template>
                    <el-menu-item :index="item2.path" v-for="(item2) in item1.children" :key="item2.id">
                        <i class="el-icon-menu"></i>
                        {{item2.authName}}
                    </el-menu-item>
                </el-submenu>

            </el-menu>
        </el-aside>
        <el-main class="main">
            <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>

</template>

<script>
export default {
  data () {
    return {
      menus: []
    }
  },
  beforeMount () {
    if (!localStorage.getItem('token')) {
      this.$router.push({
        name: 'login'
      })
    }
  },
  created () {
    this.getMenus()
  },
  methods: {
    async getMenus () {
      const res = await this.$http.get(`menus`)
      const {meta: {status}, data} = res.data
      if (status === 200) {
        this.menus = data
        console.log(this.menus)
      }
    },
    // 退出
    handleLoginout () {
      localStorage.clear()
      this.$router.push({
        name: 'login'
      })
      this.$message.warning('退出成功')
    }
  }

}
</script>

<style>
    /* 布局 */
    .container{
        height: 100%;
        background:#b3c0d1;
    }
    /* .aside{
        background:red;
    } */
    .main{
        background:gray;
    }
    /* 头部 */
    .middle{
        line-height: 60px;
        text-align: center;
    }
    .loginout{
        text-decoration: none;
        line-height: 60px;
    }
</style>
