<template>
  <div class="mod-home">
    <h3>进销存管理系统</h3>
    <panel-group></panel-group>
    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
      <line-chart :chart-data="lineChartData"/>
    </el-row>
  </div>
</template>

<script>
import lineChart from './components/lineChart'
import panelGroup from './components/panelGroup'
export default {
  components: {
    panelGroup,
    lineChart
  },
  data () {
    return {
      lineChartData: []
    }
  },
  created () {
    this.$http({
      url: this.$http.adornUrl('/admin/customer/orders/getMoney'),
      method: 'get'
    }).then(({data}) => {
      if (data && data.code === 200) {
        this.lineChartData = data.money
      } else {
        this.$message.error(data.msg)
      }
    })
  }
}
</script>

<style>
  .mod-home {
    line-height: 1.5;
  }
</style>
