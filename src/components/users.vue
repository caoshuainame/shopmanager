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
                <el-input @clear="getAllUsers()" clearable class="searchInput" placeholder="请输入内容" v-model="query">
                    <el-button @click ='searchUsers()' slot="append" icon="el-icon-search"></el-button>
                </el-input>
                <el-button type="primary" @click="showDiaAddUser()">添加用户</el-button>
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
            <el-table-column prop label="用户状态" width="120">
                <template slot-scope="scope">
                    <el-switch
                        v-model="scope.row.mg_state"
                        active-color="#13ce66"
                        inactive-color="#ff4949">
                    </el-switch>
                </template>
            </el-table-column>

            <el-table-column prop label="操作" width="200">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" circle size="mini" plain></el-button>
                    <el-button @click="showMsgBox(scope.row)" type="danger" icon="el-icon-delete" circle size="mini" plain></el-button>
                    <el-button type="success" icon="el-icon-check" circle size="mini" plain></el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页 -->
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pagenum"
            :page-sizes="[2, 4, 6, 8]"
            :page-size="2"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
        </el-pagination>
        <!-- 对话框-添加用户对话框 -->
        <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
          <el-form label-position="lift" label-width="80px" :model="formdata">
              <el-form-item label="用户名">
                <el-input v-model="formdata.username"></el-input>
              </el-form-item>
              <el-form-item label="密码">
                <el-input v-model="formdata.password"></el-input>
              </el-form-item>
              <el-form-item label="邮箱">
                <el-input v-model="formdata.email"></el-input>
              </el-form-item>
              <el-form-item label="电话">
                <el-input v-model="formdata.mobile"></el-input>
              </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
            <el-button type="primary" @click="addUser()">确 定</el-button>
          </div>
        </el-dialog>
    </el-card>

</template>

<script>
export default {
  data () {
    return {
      query: '',
      pagenum: 1,
      pagesize: 3,
      total: -1,
      list: [],
      dialogFormVisibleAdd: false,
      formdata: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }
    }
  },
  created () {
    this.getTableData()
  },
  methods: {
    //   删除-弹出确认框
    showMsgBox (user) {
      this.$confirm('是否把我删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
        // 发送请求
          const res = await this.$http.delete(`users/${user.id}`)
          console.log(res)

          const {meta: {msg, status}} = res.data
          if (status === 200) {
            // 提示成功
            this.$message.success(msg)
            this.pagenum = 1
            this.getTableData()
          }
        })
        .catch(() => {
          this.$message.info('已取消删除')
        })
    },
    //   添加用户-发送请求
    async addUser () {
      // 获取表单数据，发送内容
      const res = await this.$http.post(`users`, this.formdata)
      console.log(res)
      // 关闭对话框
      this.dialogFormVisibleAdd = false
      // 更新表格
      this.getTableData()
    },
    //   添加用户 - 显示对话框
    showDiaAddUser () {
        this.formdata = {}
      this.dialogFormVisibleAdd = true
    },
    //   清空时获取所有用户
    getAllUsers () {
      this.getTableData()
    },
    //   搜索用户
    searchUsers () {
      this.pagenum = 1
      this.getTableData()
    },

    //   分页相关的方法
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.pagesize = val
      this.getTableData()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.pagenum = val
      this.getTableData()
    },

    async getTableData () {
      const AUTH_TOKEN = localStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN
      const res = await this.$http.get(`users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`)
      console.log(res)
      const {data, meta: {status}} = res.data
      if (status === 200) {
        this.total = data.total
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
