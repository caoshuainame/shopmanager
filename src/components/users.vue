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

        <el-table v-loading="loading" :data="list" style="width: 100%">
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
                      @change="changeState(scope.row)"
                      v-model="scope.row.mg_state"
                      active-color="#13ce66"
                      inactive-color="#ff4949">
                    </el-switch>
                </template>
            </el-table-column>

            <el-table-column prop label="操作" width="200">
                <template slot-scope="scope">
                    <el-button @click="showDiaEditUser(scope.row)" type="primary" icon="el-icon-edit" circle size="mini" plain></el-button>
                    <el-button @click="showMsgBox(scope.row)" type="danger" icon="el-icon-delete" circle size="mini" plain></el-button>
                    <el-button @click="showDiaSetRole(scope.row)" type="success" icon="el-icon-check" circle size="mini" plain></el-button>
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
        <!-- 对话框-编辑 -->
        <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
          <el-form label-position="lift" label-width="80px" :model="formdata">
              <el-form-item label="用户名">
                <el-input disabled v-model="formdata.username"></el-input>
              </el-form-item>
              <el-form-item label="邮箱">
                <el-input v-model="formdata.email"></el-input>
              </el-form-item>
              <el-form-item label="电话">
                <el-input v-model="formdata.mobile"></el-input>
              </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
            <el-button type="primary" @click="editUser()">确 定</el-button>
          </div>
        </el-dialog>
        <!-- 对话框-分配角色 -->
        <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRole">
          <el-form :model="formdata" label-position="left" label-width="80px">
            <el-form-item label="用户名">{{formdata.username}}</el-form-item>
            <el-form-item label="角色">
              <el-select v-model="selectVal" placeholder="请选择角色名称">
                <el-option label="请选择" :value="-1"></el-option>
                <el-option :label="item.roleName" :value="item.id" v-for="(item) in roles" :key="item.id"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleRole = false">取 消</el-button>
            <el-button type="primary" @click="setRole()">确 定</el-button>
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
      pagesize: 2,
      total: -1,
      list: [],
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleRole: false,
      formdata: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      selectVal: -1,
      roles: [],
      currUserId: -1,
      loading: true
    }
  },
  created () {
    this.getTableData()
  },
  methods: {
    // 分配角色-发送请求
    async setRole () {
      const res = await this.$http.put(`users/${this.currUserId}/role`, {rid: this.selectVal})
      const {meta: {msg, status}} = res.data
      if (status === 200) {
        this.dialogFormVisibleRole = false
        this.$message.success(msg)
      }
    },
    // 分配角色-显示对话框
    async showDiaSetRole (user) {
      this.currUserId = user.id
      this.formdata = user
      this.dialogFormVisibleRole = true
      const res = await this.$http.get(`roles`)
      console.log(res)
      const {meta: {status}, data} = res.data
      if (status === 200) {
        this.roles = data
        console.log(this.roles)
      }
      // 获取当前用户的角色id
      const res2 = await this.$http.get(`users/${user.id}`)
      console.log(res2)
      // const {meta: {status2},data2} = res2.data
      // if (status2 === 200) {
      this.selectVal = res2.data.data.rid
      // }
    },
    // 修改用户状态
    async changeState (user) {
      const res = await this.$http.put(`users/${user.id}/state/${user.mg_state}`)
      console.log(res)
      const {meta: {status, msg}} = res.data
      if (status === 200) {
        this.$message.success(msg)
      }
    },
    //   编辑-发送数据
    async editUser () {
      const res = await this.$http.put(`users/${this.formdata.id}`, this.formdata)
      console.log(res)
      const {meta: {status}} = res.data
      if (status === 200) {
        this.dialogFormVisibleEdit = false
        this.getTableData()
      }
    },
    //   编辑-显示对话框
    async showDiaEditUser (user) {
      // this.formdata = user
      const res = await this.$http.get(`users/${user.id}`)
      this.formdata = res.data.data
      this.dialogFormVisibleEdit = true
    },
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
      this.pagenum = 1
      this.pagesize = val
      this.getTableData()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.pagenum = val
      this.getTableData()
    },

    async getTableData () {
      const res = await this.$http.get(`users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`)
      console.log(res)
      const {data, meta: {status}} = res.data
      if (status === 200) {
        this.total = data.total
        this.list = data.users
        this.loading = false
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
