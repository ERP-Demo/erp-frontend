<template>
  <el-tabs v-model="activeName" type="card" @tab-click="handleClick">
    <el-tab-pane label="检查缴费" name="first">
      <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
          <el-form-item>
            <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="getDataList()">查询</el-button>
            <el-button  type="primary" @click="Pay(),dialogVisible = true">缴费</el-button>
<!--            <el-button v-if="isAuth('requirements:requirements:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>-->
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
                  prop="patientDetailed.patientName"
                  header-align="center"
                  align="center"
                  label="患者姓名">
          </el-table-column>
          <el-table-column
                  prop="testSynthesize.testSynthesizeName"
                  header-align="center"
                  align="center"
                  label="项目编码">
          </el-table-column>
          <el-table-column
                  prop="registerId"
                  header-align="center"
                  align="center"
                  label="就诊编号">
          </el-table-column>
          <el-table-column
                  prop="testSynthesize.testSynthesizePrice"
                  header-align="center"
                  align="center"
                  label="价格">
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
<!--          <el-table-column-->
<!--                  fixed="right"-->
<!--                  header-align="center"-->
<!--                  align="center"-->
<!--                  width="150"-->
<!--                  label="操作">-->
<!--            <template slot-scope="scope">-->
<!--              <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>-->
<!--              <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>-->
<!--            </template>-->
<!--          </el-table-column>-->
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
                    prop="testSynthesize.testSynthesizeId"
                    label="项目编号"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="testSynthesize.testSynthesizeName"
                    label="项目名称"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="testSynthesize.testSynthesizePrice"
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
    </el-tab-pane>
<!--    处置缴费-->
    <el-tab-pane label="处置缴费" name="second">
      <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
          <el-form-item>
            <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="getDataList()">查询</el-button>
            <el-button  type="primary" @click="Pay1(),dialogVisible1 = true">缴费</el-button>
            <!--            <el-button v-if="isAuth('requirements:requirements:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>-->
          </el-form-item>
        </el-form>
        <el-table
                :data="dataList1 "
                border
                v-loading="dataListLoading"
                @selection-change="selectionChangeHandle1"
                style="width: 100%;">
          <el-table-column
                  type="selection"
                  header-align="center"
                  align="center"
                  width="50">
          </el-table-column>
          <el-table-column
                  prop="patientHandleApply.registerId"
                  header-align="center"
                  align="center"
                  label="就诊号">
          </el-table-column>
          <el-table-column
                  prop="patientHandle.handleName"
                  header-align="center"
                  align="center"
                  label="处置名称">
          </el-table-column>
          <el-table-column
                  prop="patientHandle.handlePrice"
                  header-align="center"
                  align="center"
                  label="处置价格">
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
          <!--          <el-table-column-->
          <!--                  fixed="right"-->
          <!--                  header-align="center"-->
          <!--                  align="center"-->
          <!--                  width="150"-->
          <!--                  label="操作">-->
          <!--            <template slot-scope="scope">-->
          <!--              <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>-->
          <!--              <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>-->
          <!--            </template>-->
          <!--          </el-table-column>-->
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
                :visible.sync="dialogVisible1"
                width="50%"
                :before-close="handleClose">
          <el-table
                  ref="multipleTable"
                  :data="tableData1"
                  tooltip-effect="dark"
                  style="width: 100%"
                  @selection-change="handleSelectionChange">
            <el-table-column
                    prop="patientHandleApply.registerId"
                    header-align="center"
                    align="center"
                    label="就诊号">
            </el-table-column>
            <el-table-column
                    prop="patientHandle.handleName"
                    header-align="center"
                    align="center"
                    label="处置名称">
            </el-table-column>
            <el-table-column
                    prop="patientHandle.handlePrice"
                    header-align="center"
                    align="center"
                    label="处置价格">
            </el-table-column>
          </el-table>
          <el-tag type="primary">项目金额总计:</el-tag>
          <el-tag type="warning">{{money1}} 元</el-tag>
          <span slot="footer" class="dialog-footer">
    <el-button @click="closeAll()">取 消</el-button>
    <el-button type="primary" @click="determine1(),dialogVisible1 = false">确 定</el-button>
  </span>
        </el-dialog>
      </div>
    </el-tab-pane>
<!--    成药缴费-->
    <el-tab-pane label="成药缴费" name="third">
      <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
          <el-form-item>
            <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="getDataList()">查询</el-button>
            <el-button  type="primary" @click="Pay2(),dialogVisible2 = true">缴费</el-button>
            <!--            <el-button v-if="isAuth('requirements:requirements:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>-->
          </el-form-item>
        </el-form>
        <el-table
                :data="dataList2"
                border
                v-loading="dataListLoading"
                @selection-change="selectionChangeHandle2"
                style="width: 100%;">
          <el-table-column
                  type="selection"
                  header-align="center"
                  align="center"
                  width="50">
          </el-table-column>
          <el-table-column
                  prop="prescription.prescriptionId"
                  header-align="center"
                  align="center"
                  label="处方编号">
          </el-table-column>
          <el-table-column
                  prop="prescription.prescriptionName"
                  header-align="center"
                  align="center"
                  label="处方名称">
          </el-table-column>
          <el-table-column
                  prop="prescription.prescriptionPrice"
                  header-align="center"
                  align="center"
                  label="处方价格">
          </el-table-column>
          <el-table-column
                  align="center"
                  label="状态"
                  show-overflow-tooltip>
            <template slot-scope="scope">
              <el-tag type="warning" v-if="scope.row.prescription.prescriptionState===-1">未开立</el-tag>
              <el-tag type="danger" v-if="scope.row.prescription.prescriptionState===0">已作废</el-tag>
              <el-tag type="info" v-if="scope.row.prescription.prescriptionState===1">未缴费</el-tag>
              <el-tag type="success" v-if="scope.row.prescription.prescriptionState===4">已续费</el-tag>
            </template>
          </el-table-column>
          <!--          <el-table-column-->
          <!--                  fixed="right"-->
          <!--                  header-align="center"-->
          <!--                  align="center"-->
          <!--                  width="150"-->
          <!--                  label="操作">-->
          <!--            <template slot-scope="scope">-->
          <!--              <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>-->
          <!--              <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>-->
          <!--            </template>-->
          <!--          </el-table-column>-->
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
                :visible.sync="dialogVisible2"
                width="73%"
                :before-close="handleClose">
          <el-table
                  ref="multipleTable"
                  :data="tableData2"
                  tooltip-effect="dark"
                  style="width: 100%"
                  @selection-change="handleSelectionChange">
            <el-table-column
                    prop="drugsDetailed.drugsId"
                    label="项目编号"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="drugsDetailed.drugsName"
                    label="药品名称"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="drugsDetailed.drugsPrice"
                    label="药品价格"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="drugsDetailed.drugsNorms"
                    label="药品规格"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="drugsDetailed.drugsDosage"
                    label="药品用量"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="drugsDetailed.drugsApprovalNumber"
                    label="批准文号"
                    width="180">
            </el-table-column>
          </el-table>
          <el-tag type="primary">项目金额总计:</el-tag>
          <el-tag type="warning">{{money2}} 元</el-tag>
          <span slot="footer" class="dialog-footer">
    <el-button @click="closeAll()">取 消</el-button>
    <el-button type="primary" @click="determine2(),dialogVisible2 = false">确 定</el-button>
  </span>
        </el-dialog>
      </div>
    </el-tab-pane>
  </el-tabs>
</template>

<script>
import AddOrUpdate from './requirements-add-or-update'
export default {
  data () {
    return {
      activeName: 'first',
      dataForm: {
        key: ''
      },
      registerId:"",
      dataA:[],
      dataList: [],
      dataList1:[],
      dataList2:[],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      dataListSelections1: [],
      dataListSelections2: [],
      addOrUpdateVisible: false,
      dialogVisible: false,
      dialogVisible1: false,
      dialogVisible2: false,
      tableData: [],
      tableData2: [],
      money:0.0,
      money1:0.0,
      money2:0.0,
      multipleSelection: [],
    }
  },
  components: {
    AddOrUpdate
  },
  activated () {
    this.getDataList()
    this.getDataList1()
    this.getDataList2()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/requirements/requirements/All'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'name': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList = data.list
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    getDataList1() {
      this.$http({
        url: this.$http.adornUrl('/patient_handle/handle/apply/payment'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'key': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList1 = data.list
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    getDataList2 () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/prescription/prescription/All'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'name': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList2 = data.list
          // this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })

    },
    closeAll(){
      this.money=null
      this.money1=null
      this.money2=null
      this.tableData2=null
      this.tableData=null
      this.tableData1=null
      this.dialogVisible1 = false
      this.dialogVisible2 = false
    },
    handleClose(done) {
      this.money=null
      this.money1=null
      this.tableData2=null
      this.tableData=null
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
    handleClick(tab, event) {
      console.log(tab, event);
    },
    //缴费
    Pay1(){
     this.tableData1=this.dataListSelections1
      this.tableData1.filter(item => {
        this.money1 += item.patientHandle.handlePrice
      })
    },
    Pay2(){
      this.tableData2=this.dataListSelections2
      this.tableData2.filter(item => {
        this.money2 += item.drugsDetailed.drugsPrice
      })
    },
    Pay() {
      this.tableData=this.dataListSelections
      this.dataListSelections.filter(item => {
        this.money += item.testSynthesize.testSynthesizePrice
      })
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    determine(){
      var ids = this.dataListSelections.map(item => {
        return item.testSynthesize.testSynthesizeId
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
    determine1(){
      var ids = this.dataListSelections1.map(item => {
        return item.patientHandle.id
      }).join(",")
      this.$http({
        url: this.$http.adornUrl('/patient_handle/handle/updatestate/'+ids),
        method: 'get',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1000,
            onClose: () => {
              this.getDataList1()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    determine2(){
      var ids = this.dataListSelections2.map(item => {
        return item.prescription.prescriptionId
      }).join(",")
      alert(ids)
      this.$http({
        url: this.$http.adornUrl(`/prescription/prescription/updatestate/`+ids),
        method: 'get',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1000,
            onClose: () => {
              this.getDataList2()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    // 多选
    selectionChangeHandle (val) {
      this.money=null
      this.money1=null
      this.dataListSelections = val
      console.log(val.testSynthesizePrice);
    },
    selectionChangeHandle2 (val) {
      this.dataListSelections2 = val
    },
    selectionChangeHandle1 (val) {
      this.dataListSelections1 = val
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
