<template>
  <div class="mod-home">
    <h3>医疗管理系统</h3>
    <panel-group></panel-group>
    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;width: 80%;display: inline-block;">
      <line-chart :chart-data="lineChartData"/>
    </el-row>
    <div class="task">
      <el-row :gutter="40" style="width: 80%;display: inline-block;" class="panel-group">
      <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
        <div class="card-panel" @click="handleSetLineChartData('shoppings')">
          <div class="card-panel-icon-wrapper icon-shopping">
            <icon-svg name="articles" class-name="card-panel-icon" />
          </div>
          <div class="card-panel-description">
            <div class="card-panel-text">任务列表</div>
<!--            <count-to :start-val="0" :end-val="13600" :duration="3600" class="card-panel-num"/>-->
          </div>
        </div>
      </el-col>
      </el-row>
    </div>
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
  }
}
</script>

<style  rel="stylesheet/scss" lang="scss" scoped>
  .mod-home {
    line-height: 1.5;
  }
  .task{
    /*border: 1px solid rgb(0, 0, 0);*/
    width: 20%;
    height: 500px;
    position: absolute;
    top: 14%;
    right: 3%;
  }
  .panel-group {
    margin-top: 18px;
    .card-panel-col{
      margin-bottom: 32px;
    }
    .card-panel {
      width: 252px;
      margin-top: -3px;
      height: 483px;
      cursor: pointer;
      font-size: 12px;
      position: relative;
      overflow: hidden;
      color: #666;
      background: #fff;
      box-shadow: 4px 4px 40px rgba(0, 0, 0, .1);
      border-color: rgba(0, 0, 0, .1);
      &:hover {
        .card-panel-icon-wrapper {
          color: #fff;
        }
        .icon-people {
          background: #40c9c6;
        }
        .icon-message {
          background: #36a3f7;
        }
        .icon-money {
          background: #f4516c;
        }
        .icon-shopping {
          background: #34bfa3
        }
      }
      .icon-people {
        color: #40c9c6;
      }
      .icon-message {
        color: #36a3f7;
      }
      .icon-money {
        color: #f4516c;
      }
      .icon-shopping {
        color: #34bfa3
      }
      .card-panel-icon-wrapper {
        float: left;
        margin: 14px 0 0 14px;
        padding: 16px;
        transition: all 0.38s ease-out;
        border-radius: 6px;
      }
      .card-panel-icon {
        float: left;
        font-size: 48px;
      }
      .card-panel-description {
        float: right;
        font-weight: bold;
        margin: 26px;
        margin-left: 0px;
        .card-panel-text {
          line-height: 18px;
          color: rgba(0, 0, 0, 0.45);
          font-size: 16px;
          margin-bottom: 12px;
        }
        .card-panel-num {
          font-size: 20px;
        }
      }
    }
  }
</style>
