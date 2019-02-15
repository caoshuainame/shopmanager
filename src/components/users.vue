<template>
<!-- 面包屑 -->
    <el-card class="box">
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

<!-- 搜索框 -->
        <el-row class="searchBox">
            <el-col>
                <el-input class="searchInput" placeholder="请输入内容" v-model="query">
                    <el-button slot="append" icon="el-icon-search"></el-button>
                </el-input>
                <el-button type="primary">添加用户</el-button>
            </el-col>
        </el-row>
<!-- 表格 -->
      <!-- id: 500
      username: "admin"
      email: "adsfad@qq.com"
      mobile: "12345678"
      create_time: 1486720211
      mg_state: true
      role_name: "主管" -->
        <el-table :data="list" style="width: 100%">
            <el-table-column prop="id" label="#" width="80"></el-table-column>
            <el-table-column prop="username" label="姓名" width="100"></el-table-column>
            <el-table-column prop="email" label="邮箱" width="140"></el-table-column>
            <el-table-column prop="mobile" label="电话" width="140"></el-table-column>
            <el-table-column label="创建日期" width="200">
                <template slot-scope="list">
                    {{list.row.create_time|fmtdate}}
                </template>
            </el-table-column>
            <el-table-column prop="name" label="用户状态" width="120"></el-table-column>
            <el-table-column prop="name" label="操作" width="200"></el-table-column>
        </el-table>
        </el-card>

</template>

<script>
export default {
  data () {
    return {
      query: '',
      pagenum: 1,
      pagesize: 10,
      list: []
    }
  },
  created () {
    this.getTabData()
  },
  methods: {
    async getTabData () {
      const AUTH_TOKEN = localStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN
      const res = await this.$http.get(`users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`)
      console.log(res)
      const {data, meta: {status, msg}} = res.data
      if (status === 200) {
        this.list = data.users
      }
    }
  }
}
</script>

<style>
    .searchBox{
        margin-top: 20px;
    }
    .searchInput{
        width: 400px;
    }
</style>
