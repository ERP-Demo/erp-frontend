<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" >
      <el-form-item label="状态：" >
        <el-select placeholder="请选择状态" v-model="dataForm.prescriptionState" clearable style="width: 130px" class="filter-item">
          <el-option v-for="item in [{key:0,value:'未取药'},{key:1,value:'已取药'}]" :key="item.key" :label="item.value" :value="item.key" />
        </el-select>
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
        prop="username"
        header-align="center"
        align="center"
        label="医生">
    </el-table-column>
    <el-table-column
        prop="patientName"
        header-align="center"
        align="center"
        label="患者">
    </el-table-column>
    <el-table-column
        prop="prescriptionName"
        header-align="center"
        align="center"
        label="处方名称">
    </el-table-column>
    <el-table-column
        prop="prescriptionPrice"
        header-align="center"
        align="center"
        label="应付金额">
    </el-table-column>
    <el-table-column
        prop="prescriptionState"
        header-align="center"
        align="center"
        label="处方状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.prescriptionState===0" type="danger" >未取药</el-tag>
          <el-tag v-if="scope.row.prescriptionState===1" type="primary">已取药</el-tag>
        </template>
    </el-table-column>
    <el-table-column
      fixed="right"
      header-align="center"
      align="center"
      width="160"
      label="操作">
      <template slot-scope="scope">
        <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.prescriptionId)" v-if="scope.row.prescriptionState===0" >取药</el-button>
        <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.prescriptionId)" v-if="scope.row.prescriptionState===1" disabled="dis" >取药</el-button>
        <el-button type="text" size="small" @click="dialogVisible1 = true,toviwe(scope.row.prescriptionId)">详情</el-button>
      </template>
    </el-table-column>
    </el-table>
    <el-dialog
            title="提示"
            :visible.sync="dialogVisible1"
            width="66%"
            :before-close="handleClose">
      <el-table
              ref="multipleTable"
              :data="pdData"
              tooltip-effect="dark"
              style="width: 100%"
              @selection-change="handleSelectionChange">
        <el-table-column
                prop="pdId"
                header-align="center"
                align="center"
                label="处方详情编号">
        </el-table-column>
        <el-table-column
                prop="prescriptionName"
                header-align="center"
                align="center"
                label="处方名称">
        </el-table-column>
        <el-table-column
                prop="drugsName"
                header-align="center"
                align="center"
                label="药品名称">
        </el-table-column>
        <el-table-column
                prop="drugsNum"
                header-align="center"
                align="center"
                label="药品数量">
        </el-table-column>
        <el-table-column
                prop="drugsUse"
                header-align="center"
                align="center"
                label="使用方法">
        </el-table-column>
        <el-table-column
                prop="drugsDay"
                header-align="center"
                align="center"
                label="天数">
        </el-table-column>
        <el-table-column
                prop="drugsUsenum"
                header-align="center"
                align="center"
                label="用量">
        </el-table-column>
        <el-table-column
                prop="status"
                header-align="center"
                align="center"
                label="状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status===2" type="primary">已缴费</el-tag>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible1 = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible1 = false">确 定</el-button>
  </span>
    </el-dialog>

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
        prescriptionState:null
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
  components: {
    // AddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/prescription/prescription/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'prescriptionState': this.dataForm.prescriptionState
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
    addHandle(id){
      this.pid=id;
    },
    //多表数据
    toviwe(id){
      console.log(this.dataList)
      this.dataList.forEach(item =>{
        if(item.prescriptionId===id){
          this.pdData=item.prescriptionDetails
        }
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
      this.dataList.forEach(item =>{
        if(item.prescriptionId===id){
          this.pdData=item.prescriptionDetails
          // var nums = this.pdData.drugsNum ? [this.pdData.drugsNum] : this.pdData.map(p => {
          //   this.pdData.forEach(pitem =>{
          //     return pitem.drugsNum
          //   })
          //   return p.drugsNum
          // })
          // console.log(nums)
          this.$http({
              url: this.$http.adornUrl('/prescription/prescription/bypdid/'+id),
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
      })
      // if (sta === 2) {
      //   this.$message({
      //     message: '请先缴费',
      //     type: 'warning',
      //     duration: 1000,
      //   })
      // } else {
        // this.$http({
        //   url: this.$http.adornUrl('/prescription/prescription/bypdid/'+id),
        //   method: 'get'
        // }).then(({data}) => {
        //   if (data && data.code === 200) {
        //     this.$message({
        //       message: '操作成功',
        //       type: 'success',
        //       duration: 1000,
        //       onClose: () => {
        //         this.getDataList()
        //       }
        //     })
        //   } else {
        //     this.$message.error(data.msg)
        //   }
        // })
      // }
    },
    //获取药
    getData () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/drugs/detailed/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'name': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.tableData = data.page.list
        } else {
          this.dataList = []
        }
        this.dataListLoading = false
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
          url: this.$http.adornUrl('/supplier/detailed/delete'),
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
