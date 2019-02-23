<template>
    <el-card class="box-card">
        <!-- 面包屑 -->
        <cus-bread level1="数据统计" level2="数据报表"></cus-bread>
        <!-- 图标容器 -->
        <div id="main" style="width: 600px;height:400px;"></div>

    </el-card>
</template>

<script>
import MyEcharts from 'echarts'

export default {
  data () {
    return {

    }
  },
  mounted () {
    this.initEcharts()
  },
  methods: {
    async initEcharts () {
      const myChart = MyEcharts.init(document.getElementById('main'))
      const res = await this.$http.get(`reports/type/1`)
      console.log(res)
      const option2 = res.data.data
      // 绘制图表
      const option1 = {
        title: {
          text: '我是图示'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#6a7985'
            }
          }
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        }
      }
      const option = {...option2, ...option1}
      myChart.setOption(option)
    }
  }
}
</script>

<style>
.box-card{
    height: 100%;
}
</style>
