<template>
    <el-card class="box">
        <cus-bread level1='商品管理' level2='商品列表'></cus-bread>
        <el-alert class="alert" title="添加商品信息" type="info" center show-icon></el-alert>
        <el-steps :active="active*1" align-center>
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
        </el-steps>

        <!-- 标签页 -->
      <el-form class="form" label-position="top" label-width="80px" :model="form">
        <el-tabs v-model="active" tab-position="left" @tab-click="changeTab()">
          <el-tab-pane label="基本信息" name="1">
            <el-form-item label="商品名称">
                <el-input v-model="form.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格">
                <el-input v-model="form.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量">
                <el-input v-model="form.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量">
                <el-input v-model="form.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类">
                <el-cascader
                clearable
                  expand-trigger="hover"
                  :options="options"
                  v-model="selectedOptions"
                  @change="handleChange"
                  :props="defaultProp"
                  >
                </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="2">
            <el-form-item :label="item1.attr_name" v-for="(item1) in arrDy" :key="item1.attr_id">
              <!-- {{checkList}} -->
              <el-checkbox-group v-model ="item1.attr_vals">
                <el-checkbox border :label="item2" v-for="(item2,i) in item1.attr_vals" :key = "i"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="3">
            <el-form-item :label="item.attr_name" v-for="(item) in arrStatic" :key="item.attr_id">
                <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="4">
            <el-form-item label="添加图片">
              <el-upload
                :headers="headers"
                action="http://localhost:8888/api/private/v1/upload"
                :on-remove="handleRemove"
                :on-success="handleSuccess"
                list-type="picture">
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="5">
            <el-form-item>
              <el-button @click="addGoods()">添加商品</el-button>
              <quill-editor v-model="form.goods_introduce" class="quill"></quill-editor>
            </el-form-item>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
</template>

<script>
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import { quillEditor } from 'vue-quill-editor'

export default {
  components: {
    quillEditor
  },
  data () {
    return {
      active: '1',
      //   添加商品请求时的请求体
      form: {
        goods_name: '',
        goods_cat: '',
        goods_price: '',
        goods_number: '',
        goods_weight: '',
        goods_introduce: '',
        pics: [],
        attrs: []
      },
      //   级联配置的数组
      options: [],
      selectedOptions: [1, 3, 6],
      defaultProp: {
        label: 'cat_name',
        value: 'cat_id'
        // children: 'children'
      },
      // checkList: [],
      //   动态参数的数组
      arrDy: [],
      arrStatic: [],
      headers: {
        Authorization: localStorage.getItem('token')
      },
      flag: true
    }
  },
  created () {
    this.getGoodsCate()
  },
  methods: {
    async addGoods () {
      this.form.goods_cat = this.selectedOptions.join(',')

      // const res = this.$http.post(`goods`, this.form)
      // console.log(res)
      // 处理动态
      const obj1 = {attr_id: '', attr_vals: ''}
      const arr1 = []

      this.arrDy.forEach(item => {
        obj1.attr_id = item.attr_id
        obj1.attr_vals = item.attr_vals
        arr1.push(obj1)
      })

      // 处理静态
      const obj2 = {attr_id: '', attr_vals: ''}
      const arr2 = []

      this.arrStatic.forEach(item => {
        obj2.attr_id = item.attr_id
        obj2.attr_vals = item.attr_vals
        arr2.push(obj2)
      })

      this.form.attrs = [...arr1, ...arr2]

      const res = await this.$http.post(`goods`, this.form)

      console.log(res)
      const {meta: {msg, status}} = res.data
      if (status === 201) {
        this.$router.push({
          name: 'goods'
        })
      } else {
        this.$message.error(msg)
      }
    },
    // 移除文件
    handleRemove (file, fileList) {
      // console.log('remove-----')
      // console.log(file)
      const Index = this.form.pics.map((item) => {
        return item.pic === file.reponse.data.tmp_path
      })
      this.form.pics.splice(Index, 1)
    },
    // 假上传成功
    handleSuccess (res, file, fileList) {
      // console.log('success-----')
      // console.log(res)
      const temPath = res.data.tmp_path
      this.form.pics.push({
        pic: temPath
      })
    },
    //   点击任何tap触发
    async changeTab () {
      if (this.active === '2' || this.active === '3') {
        if (this.selectedOptions.length !== 3) {
          this.$message.error('请选择三级分类')
          if (this.active === '2') {
            this.arrDy = []
          } else {
            this.arrStatic = []
          }
          return
        }
        if (this.active === '2' && this.flag === true) {
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
              this.flag = false
            })
            // console.log(this.arrDy)
          }
        }
        if (this.active === '3') {
          const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=only`)
          //   console.log(res)
          const {meta: {status}, data} = res.data
          if (status === 200) {
            this.arrStatic = data

            // console.log(this.arrStatic)
          }
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
    },
    handleChange () {

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
    margin-bottom: 20px;
}
.form{
    height: 400px;
    overflow: auto;
}
.ql-editor,
.ql-blank{
  min-height: 200px;
}
</style>
