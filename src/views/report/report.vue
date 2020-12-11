<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/homr' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
       <div id="main" style="width: 750px;height:400px;"></div>
    </el-card>
  </div>
</template>
<script>
import _ from 'lodash'
const echarts = require('echarts')
export default {

  data () {
    return {
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  created () {},
  async mounted () {
    var myChart = echarts.init(document.getElementById('main'))

    // 指定图表的配置项和数据
    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) return this.$message.error('获取报表数据失败')
    console.log(res)
    // 使用刚指定的配置项和数据显示图表。
    const options = _.merge(res.data, this.options)
    myChart.setOption(options)
  },
  methods: {}
}
</script>
<style lang="less" scoped></style>
