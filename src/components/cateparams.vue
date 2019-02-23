<template>
    <el-card class="box">
        <cus-bread level1='商品管理' level2='分类参数'></cus-bread>
        <el-alert class="alert" title="注意：只允许为第三级分类设置参数" type="warning" show-icon></el-alert>
        <!-- 级联 -->
        <el-form class="form" label-position="left" label-width="160px" :model="form">
            <el-form-item label="请选择商品分类">
              {{selectedOptions}}
              <el-cascader
                clearable
                expand-trigger="hover"
                :options="options"
                v-model="selectedOptions"
                @change="handleChange"
                :props="defaultProp"
                ></el-cascader>
            </el-form-item>
        </el-form>
        <!-- 横导航  -->
        <el-tabs type="border-card" v-model="active" @tab-click="changeTab()">
          <el-tab-pane name='1' label="动态参数">
            <el-button disabled>设置动态参数</el-button>
            <!-- 表格 -->
            <el-table height="350px" border stripe :data="arrDy" style="width: 100%">
                <!-- 序号 -->
                <el-table-column type="expand" width="140">
                  <template slot-scope="scope">
                    <el-tag
                      :key="i"
                      v-for="(attr,i) in scope.row.attr_vals"
                      closable
                      :disable-transitions="false"
                      @close="handleClose(scope.row,attr)">
                      {{attr}}
                    </el-tag>
                    <el-input
                      class="input-new-tag"
                      v-if="inputVisible"
                      v-model="inputValue"
                      ref="saveTagInput"
                      size="small"
                      @keyup.enter.native="handleInputConfirm(scope.row)"
                      @blur="handleInputConfirm(scope.row)"
                    >
                    </el-input>
                    <el-button v-else class="button-new-tag" size="small" @click="showInput">+ New Tag</el-button>

                  </template>
                </el-table-column>
                <el-table-column type="index" label='#' width="120"> </el-table-column>
                <el-table-column prop="attr_name" label="属性名称" width="240"></el-table-column>

                <el-table-column label="操作" width="300">
                  <template slot-scope="scope">
                    <el-button  plain size="mini" type="primary" icon="el-icon-edit" circle></el-button>
                    <el-button  plain size="mini" type="danger" icon="el-icon-delete" circle></el-button>
                  </template>
                </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane name='2' label="静态参数">
            <el-button disabled>设置静态参数</el-button>
            <el-table height="450px" border stripe :data="arrStatic" style="width: 100%">
                <el-table-column type="index" label='#' width="120"> </el-table-column>
                <el-table-column prop="attr_name" label="属性名称" width="240"></el-table-column>
                <el-table-column prop="attr_vals" label="属性值" width="240"></el-table-column>

                <el-table-column label="操作" width="300">
                  <template slot-scope="scope">
                    <el-button  plain size="mini" type="primary" icon="el-icon-edit" circle></el-button>
                    <el-button  plain size="mini" type="danger" icon="el-icon-delete" circle></el-button>
                  </template>
                </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
    </el-card>
</template>

<script>
export default {
  data () {
    return {
      active: '1',
      form: {},
      //   级联配置的数组
      options: [],
      selectedOptions: [],
      defaultProp: {
        label: 'cat_name',
        value: 'cat_id'
        // children: 'children'
      },
      arrDy: [],
      arrStatic: [],
      // 动态tag编辑用的数据
      // dynamicTags: ['标签一', '标签二', '标签三'],
      inputVisible: false,
      inputValue: '',
      inputVals: {}
    }
  },
  created () {
    this.getGoodsCate()
  },
  methods: {
    async handleClose (item, attr) {
      item.attr_vals.splice(item.attr_vals.indexOf(attr), 1)
      // console.log(item.attr_vals)
      // arrDy每个元素的attr_vals
      const res = await this.$http.put(`categories/${this.selectedOptions[2]}/attributes/${item.attr_id}`, {
        attr_name: item.attr_name,
        attr_sel: 'many',
        attr_vals: item.attr_vals.join(',')
      })
      console.log(res)
    },

    showInput () {
      this.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },

    async handleInputConfirm (item) {
      let inputValue = this.inputValue
      if (inputValue) {
        item.attr_vals.push(inputValue)
        const res = await this.$http.put(`categories/${this.selectedOptions[2]}/attributes/${item.attr_id}`, {
          attr_name: item.attr_name,
          attr_sel: 'many',
          attr_vals: item.attr_vals.join(',')
        })
        console.log(res)
      }
      this.inputVisible = false
      this.inputValue = ''
    },
    handleChange () {
      this.getDyOrStatic()
    },
    changeTab () {
      this.getDyOrStatic()
    },
    // 获取动态、静态数据
    async getDyOrStatic () {
      // console.log('级联被label改变')
      if (this.selectedOptions.length !== 3) {
        this.$message.warning('请先写第三级分类设置参数')
        if (this.active === '1') {
          this.arrDy = []
        }
        if (this.active === '2') {
          this.arrStatic = []
        }
        return
      }
      if (this.active === '1') {
        const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=many`)
        //   console.log(res)
        const {meta: {status}, data} = res.data
        if (status === 200) {
          this.arrDy = data
          this.arrDy.forEach(item => {
            if (item.attr_vals.length === 0) {
              item.attr_vals = []
            } else {
              item.attr_vals = item.attr_vals.trim().split(',')
            }
          })
          console.log(this.arrDy)
        }
      }
      if (this.active === '2') {
        const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=only`)
        //   console.log(res)
        const {meta: {status}, data} = res.data
        if (status === 200) {
          this.arrStatic = data

          console.log(this.arrStatic)
        }
      }
    },
    //   获取三级分类的数据
    async getGoodsCate () {
      const res = await this.$http.get(`categories?type=3`)
      const {meta: {status}, data} = res.data
      if (status === 200) {
        this.options = data
        // console.log(this.options)
      }
    }
  }
}
</script>

<style>
.box{
    height: 100%;
}
.alert{
    margin-top: 20px;
}
.el-tag + .el-tag {
    margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
