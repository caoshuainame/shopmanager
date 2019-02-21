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
          <el-tab-pane label="商品属性" name="3">商品属性</el-tab-pane>
          <el-tab-pane label="商品图片" name="4">商品图片</el-tab-pane>
          <el-tab-pane label="商品内容" name="5">商品内容</el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
</template>

<script>
export default {
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
        pics: '',
        attrs: ''
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
      arrStatic:[]
    }
  },
  created () {
    this.getGoodsCate()
  },
  methods: {
    //   点击任何tap触发
    async changeTab () {
      if (this.active === '2' || this.active === '3') {
        if (this.selectedOptions.length !== 3) {
          this.$message.error('请选择三级分类')
          return
        }
        if (this.active === '2') {
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
        if (this.active==='3'){
          const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=only`)
          //   console.log(res)
          const {meta: {status}, data} = res.data
          if (status === 200) {
            this.arrStatic = data
            
            console.log(this.arrStatic)
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
</style>
