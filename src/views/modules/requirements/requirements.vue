<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button  type="primary" :disabled="dataListSelections.length >= 2" @click="Pay(),dialogVisible = true">缴费</el-button>
        <el-button v-if="isAuth('requirements:requirements:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
    <el-table-column
        prop="projectId"
        header-align="center"
        align="center"
        label="项目编码">
    </el-table-column>
    <el-table-column
        prop="projectName"
        header-align="center"
        align="center"
        label="项目名称">
    </el-table-column>
    <el-table-column
        prop="purpose"
        header-align="center"
        align="center"
        label="目的">
    </el-table-column>
    <el-table-column
        prop="requirements"
        header-align="center"
        align="center"
        label="要求">
    </el-table-column>
    <el-table-column
        prop="clinicalImpression"
        header-align="center"
        align="center"
        label="临床印象">
    </el-table-column>
    <el-table-column
        prop="clinicalDiagnosis"
        header-align="center"
        align="center"
        label="临床诊断">
    </el-table-column>
    <el-table-column
        prop="checkThe"
        header-align="center"
        align="center"
        label="检查部位">
    </el-table-column>
      <el-table-column
              align="center"
              label="状态"
              show-overflow-tooltip>
        <template slot-scope="scope">
          <el-tag type="warning" v-if="scope.row.status===-1">未开立</el-tag>
          <el-tag type="danger" v-if="scope.row.status===0">已作废</el-tag>
          <el-tag type="info" v-if="scope.row.status===1">未缴费</el-tag>
          <el-tag type="success" v-if="scope.row.status===4">已续费</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
<!--          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>-->
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <el-dialog
            title="费用详细"
            :visible.sync="dialogVisible"
            width="50%"
            :before-close="handleClose">
      <el-table
              ref="multipleTable"
              :data="tableData"
              tooltip-effect="dark"
              style="width: 100%"
              @selection-change="handleSelectionChange">
        <el-table-column
                type="selection"
                width="55">
        </el-table-column>
        <el-table-column
                prop="testSynthesizeId"
                label="项目编号"
                width="180">
        </el-table-column>
        <el-table-column
                prop="testSynthesizeName"
                label="项目名称"
                width="180">
        </el-table-column>
        <el-table-column
                prop="testSynthesizePrice"
                label="项目价格">
        </el-table-column>
      </el-table>
      <el-tag type="primary">项目金额总计:</el-tag>
      <el-tag type="warning">{{money}} 元</el-tag>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="determine(),dialogVisible = false">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
import AddOrUpdate from './requirements-add-or-update'
export default {
  data () {
    return {
      dataForm: {
        key: ''
      },
      registerId:"",
      dataA:[],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      dialogVisible: false,
      tableData: [],
      money:0.0,
      multipleSelection: []
    }
  },
  components: {
    AddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/requirements/requirements/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'key': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList = data.page.list
          this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    handleClose(done) {
                done();
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    //缴费
    Pay() {
      var ids = this.dataListSelections.map(item => {
        return item.projectId
      }).join(",")
      var sta = this.dataListSelections.map(item => {
        return item.status
      }).join(",")
      if(sta==4){
        return;
      }
      this.$http({
        url: this.$http.adornUrl('/test_synthesize/synthesize/selectByid/'+ids),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.tableData=data.list
        } else {
          this.tableData=[]
        }
      })
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
      var moe = this.multipleSelection.map(item => {
        return item.testSynthesizePrice
      }).join(",")
      this.money=moe
    },
    determine(){
      var ids = this.dataListSelections.map(item => {
        return item.projectId
      }).join(",")
      this.$http({
        url: this.$http.adornUrl(`/requirements/requirements/updatestate/`+ids),
        method: 'get',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1000,
            onClose: () => {
              this.getDataList()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle (id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.id
      })
      this.$confirm(`确定对这${ids.length}条数据进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/requirements/requirements/delete'),
          method: 'delete',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  }
}
</script>
