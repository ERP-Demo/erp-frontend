<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="供应商" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getlike()">查询</el-button>
        <el-button v-if="isAuth('supplier:detailed:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('supplier:detailed:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        prop="supplierName"
        header-align="center"
        align="center"
        label="供应商名称">
    </el-table-column>
    <el-table-column
        prop="supplierCartPhone"
        header-align="center"
        align="center"
        label="联系电话">
    </el-table-column>
    <el-table-column
        prop="supplierType"
        header-align="center"
        align="center"
        label="供应商类型：1.自营，2.平台">
    </el-table-column>
    <el-table-column
        prop="supplierMan"
        header-align="center"
        align="center"
        label="供应商联系人">
    </el-table-column>
    <el-table-column
        prop="supplierBankName"
        header-align="center"
        align="center"
        label="开户银行名称">
    </el-table-column>
    <el-table-column
        prop="supplierBankAccount"
        header-align="center"
        align="center"
        label="银行账号">
    </el-table-column>
    <el-table-column
        prop="supplierAddress"
        header-align="center"
        align="center"
        label="供应商地址">
    </el-table-column>
    <el-table-column
        prop="supplierStatus"
        header-align="center"
        align="center"
        label="状态：0禁止，1启用">
      <template slot-scope="scope">
        <el-tag :type="scope.row.supplierStatus === 0 ? 'primary' : 'dangers'"
                disable-transitions>{{scope.row.supplierStatus===0 ? '正常' : '禁用'}}</el-tag>
      </template>
    </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="160"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="dialogVisible = true,addHandle(scope.row.supplierId)">新增</el-button>
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.supplierId)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.supplierId)">删除</el-button>
          <el-button type="text" size="small" @click="dialogVisible1 = true,toviwe(scope.row.supplierId)">查看</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
            title="提示"
            :visible.sync="dialogVisible"
            width="66%"
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
                prop="drugsId"
                header-align="center"
                align="center"
                label="药品编号">
        </el-table-column>
        <el-table-column
                prop="drugsName"
                header-align="center"
                align="center"
                label="药品名称">
        </el-table-column>
        <el-table-column
                prop="drugsPrice"
                header-align="center"
                align="center"
                label="药品单价">
        </el-table-column>
        <el-table-column
                prop="drugsNorms"
                header-align="center"
                align="center"
                label="药品规格">
        </el-table-column>
        <el-table-column
                prop="drugsUsage"
                header-align="center"
                align="center"
                label="药品用法">
        </el-table-column>
        <el-table-column
                prop="drugsDosage"
                header-align="center"
                align="center"
                label="药品用量">
        </el-table-column>
        <el-table-column
                prop="drugsTaboo"
                header-align="center"
                align="center"
                label="禁忌">
        </el-table-column>
        <el-table-column
                prop="drugsProducer"
                header-align="center"
                align="center"
                label="生产厂商">
        </el-table-column>
        <el-table-column
                prop="drugsApprovalNumber"
                header-align="center"
                align="center"
                label="批准文号">
        </el-table-column>
      </el-table>
      <div style="margin-top: 20px">
        <el-button @click="toggleSelection()">取消选择</el-button>
      </div>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false,confirmf()">确 定</el-button>
  </span>
    </el-dialog>
    <el-dialog
            title="提示"
            :visible.sync="dialogVisible1"
            width="66%"
            :before-close="handleClose">
      <el-table
              ref="multipleTable"
              :data="drugsData"
              tooltip-effect="dark"
              style="width: 100%"
              @selection-change="handleSelectionChange">
        <el-table-column
                type="selection"
                width="55">
        </el-table-column>
        <el-table-column
                prop="drugsId"
                header-align="center"
                align="center"
                label="药品编号">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsName"
                header-align="center"
                align="center"
                label="药品名称">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsPrice"
                header-align="center"
                align="center"
                label="药品单价">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsNorms"
                header-align="center"
                align="center"
                label="药品规格">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsUsage"
                header-align="center"
                align="center"
                label="药品用法">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsDosage"
                header-align="center"
                align="center"
                label="药品用量">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsTaboo"
                header-align="center"
                align="center"
                label="禁忌">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsProducer"
                header-align="center"
                align="center"
                label="生产厂商">
        </el-table-column>
        <el-table-column
                prop="drugsDetailed.drugsApprovalNumber"
                header-align="center"
                align="center"
                label="批准文号">
        </el-table-column>
        <el-table-column
                fixed="right"
                header-align="center"
                align="center"
                width="160"
                label="操作">
          <template slot-scope="scope">
            <el-button type="text" size="small" @click="delHandle(scope.row.drugsId)">删除</el-button>
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './detailed-add-or-update'
export default {
  data () {
    return {
      dataForm: {
        key: ''
      },
      did:0,
      checkBoxData: [],
      drugsData:[],
      dataList: [],
      dialogVisible: false,
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
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
    AddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  created(){
    this.getData()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/supplier/detailed/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'name': this.dataForm.key
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
      this.did=id
      this.$http({
        url: this.$http.adornUrl('/drugsSupplier/drugsp/all/'+id),
        method: 'get',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.code === 200) {
           this.drugsData = data.list
        } else {
          this.drugsData = []
        }
      })
    },
    //删除多表
    delHandle(id){
      this.$confirm(`确定要删除！`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/drugsSupplier/drugsp/deletebyid/'+id+"/"+this.did),
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
      })
    },
    //取消全选
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    //提交
    confirmf(){
      var ids=this.multipleSelection.map(item =>{return item.drugsId}).join(",")
      this.$http({
        url: this.$http.adornUrl('/supplier/detailed/add/'+this.pid+"/"+ids),
        method: 'get',
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
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    //获取药
    getData () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/drugs/detailed/list'),
        method: 'get',
        params: this.$http.adornParams({})
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
