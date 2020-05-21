<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" >
      <el-form-item>
        <el-input v-model="dataForm.supplierName" placeholder="患者名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
    <el-table-column
        prop="registerId"
        header-align="center"
        align="center"
        label="就诊号">
    </el-table-column>
    <el-table-column
        prop="patientName"
        header-align="center"
        align="center"
        label="患者名称">
    </el-table-column>
    <el-table-column
        prop="handleName"
        header-align="center"
        align="center"
        label="处置名称">
    </el-table-column>
    <el-table-column
        prop="handlePrice"
        header-align="center"
        align="center"
        label="处置价格">
    </el-table-column>
    <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="处方状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status===2" type="danger">未执行</el-tag>
          <el-tag v-if="scope.row.status===3" type="primary">已执行</el-tag>
        </template>
    </el-table-column>
    <el-table-column
            prop="username"
            header-align="center"
            align="center"
            label="执行人">
      <template slot-scope="scope">
        <span v-if="scope.row.username===''">暂未执行</span>
        {{scope.row.username}}
      </template>
    </el-table-column>
    <el-table-column
      fixed="right"
      header-align="center"
      align="center"
      width="160"
      label="操作">
      <template slot-scope="scope">
        <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)" v-if="scope.row.status===2">执行</el-button>
        <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)" v-if="scope.row.status===3" disabled="dis" >执行</el-button>
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
<!--    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>-->
  </div>
</template>

<script>

export default {
  data () {
    return {
      dataForm: {
        patientName:null
      },
      did:0,
      checkBoxData: [],
      pdData:[],
      dataList: [],
      dialogVisible: false,
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      sta:null,
      pid:0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      tableData: [],
      multipleSelection: [],
      dialogVisible1: false
    }
  },
  activated () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/patient_handle/handle/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'patientName': this.dataForm.patientName
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList = data.page2.list
          this.totalPage = data.page2.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })

    },


    //删除多表


    //提交

    //获取复选框
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    //关闭
    handleClose(done) {
                done();
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle (id) {
        this.$http({
          url: this.$http.adornUrl('/patient_handle/handle/runHandle/'+id),
          method: 'get'
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
    }
  }
}
</script>
