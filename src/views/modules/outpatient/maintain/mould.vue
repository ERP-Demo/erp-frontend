<template>
  <!-- 病历模板管理 -->
  <div class="div1">
    <el-container>
      <transition name="el-zoom-in-left">
        <el-aside :width="asidewidth" style="margin-top:0;background:white;padding: 0 0 0 0">
          <aside style="height:50px;margin:0 0 0 0">
            <el-tag size="large">病历模板</el-tag>
          </aside>
          <div style="padding: 0 10px 0 10px">
            <el-card v-if="!isaside">
              <el-form style="background:white" :model="listQuery" inline>
                <el-form-item label="主诉:" style="width:280px">
                  <el-input v-model="listQuery.name" placeholder="主诉"></el-input>
                </el-form-item>
                <el-form-item style="width:400px">
                  <el-button type="primary" @click="searchModel" v-if="!isaside">搜索模板</el-button>
                  <el-button type="primary" @click="showaside('add')" v-if="!isaside">新建模板</el-button>
                </el-form-item>
              </el-form>
            </el-card>
            <div style="background:white;">
              <el-table style="margin-bottom:50px" :data="dataList1">
                <el-table-column align="center" label="模板编号" prop="testModelId" width="80">
                  <template slot-scope="scope">
                    {{scope.row.tid}}
                  </template>
                </el-table-column>
                <el-table-column align="center" label="主诉" prop="testModelName">
                  <template slot-scope="scope">
                    {{scope.row.complain}}
                  </template>
                </el-table-column>
                <el-table-column align="center" label="现病史" prop="testModelIntroduction">
                  <template slot-scope="scope">
                    {{scope.row.patientSymptom}}
                  </template>
                </el-table-column>
                <el-table-column align="center" label="主要诊断" prop="testModelIntroduction">
                  <template slot-scope="scope">
                    {{scope.row.mainIcd}}
                  </template>
                </el-table-column>
                <el-table-column label="操作" prop="id" width="150px">
                  <template slot-scope="scope">
                    <el-button type="primary" size="mini" @click="editModel(scope.row.tid)">修改</el-button>
                    <el-button type="danger" size="mini"
                               @click="deleteModel(scope.row.tid)">删除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
              <pagination v-show="total>0" :total="total" :page.sync="listQuery.pageNum"
                          :limit.sync="listQuery.pageSize"/>
            </div>
          </div>
        </el-aside>
      </transition>
      <!-- 编辑模板 -->
      <el-main style="padding:0 0 0 0;" v-if="isaside">
        <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="edit" @click="updateModel">提交修改</el-button>
        <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="!edit" @click="createModel">新建模板</el-button>
        <el-button type="danger" @click="showaside">取消</el-button>
        <el-form :model="datafrom" label-width="140px" inline>
          <el-form-item label="主诉">
            <el-input placeholder="请输入主诉" v-model="datafrom.complain" style="width:1000px"></el-input>
          </el-form-item>
          <el-form-item label="现病史">
            <el-input aria-placeholder="请输入现病史" v-model="datafrom.patientSymptom" style="width:1000px"></el-input>
          </el-form-item>
          <el-form-item label="主要诊断">
            <el-input aria-placeholder="请输入主要诊断" v-model="datafrom.mainIcd" style="width:1000px"></el-input>
          </el-form-item>
        </el-form>
        <el-button type="primary" style="margin-bottom:10px;margin-left:10px" @click="addItem">编辑项目</el-button>
        <el-table style="margin-bottom:50px" :data="dataList3" v-if="!edit">
          <el-table-column label="诊断id" prop="icdId"></el-table-column>
          <el-table-column label="诊断名称" prop="icdName"></el-table-column>
          <el-table-column label="疾病编码(元)" prop="icdCode"></el-table-column>
        </el-table>
        <el-table style="margin-bottom:50px" :data="dataList4" v-if="edit">
          <el-table-column label="诊断id" prop="icdId"></el-table-column>
          <el-table-column label="诊断名称" prop="icdName"></el-table-column>
          <el-table-column label="疾病编码" prop="icdCode"></el-table-column>
          <el-table-column label="操作" prop="id" width="100px">
            <template slot-scope="scope">
              <el-button type="danger" size="mini" @click="removeList(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-main>
    </el-container>
    <el-dialog title="添加诊断" :visible.sync="dialogVisible" width="650px" top="10px">
      <el-container>
        <el-main>
          <el-button type="primary" style="float:right;margin-right: 6%" @click="addTest">确定</el-button>
          <el-table :data="icdList" cell-style="text-align:center" header-cell-style="text-align:center" style="margin-bottom: 5%" @selection-change="handleSelectionChange">
            <el-table-column type="selection" header-align="center" align="center" width="50">
            </el-table-column>
            <el-table-column property="icdId" label="诊断id" width="150"></el-table-column>
            <el-table-column property="icdName" label="诊断名称" width="150"></el-table-column>
            <el-table-column property="icdCode" label="疾病编码" width="200"></el-table-column>
          </el-table>
            <el-pagination
                    @size-change="sizeChangeHandle"
                    @current-change="currentChangeHandle"
                    :current-page="page.pageIndex"
                    :page-sizes="[10, 20, 50, 100]"
                    :page-size="page.pageSize"
                    :total="page.totalPage"
                    layout="total, sizes, prev, pager, next, jumper">
            </el-pagination>
        </el-main>
      </el-container>
    </el-dialog>
  </div>
</template>
<script>
  import Pagination from '@/components/Pagination'

  export default {
    components: {Pagination},
    data() {
      return {
        dataList1: [],
        dataList3: [],
        dataList4: [],
        icdList:[],
        page: {
          pageIndex: 1,
          pageSize: 10,
          totalPage: 0,
          dataForm: {
            key: ''
          }
        },
        oneprescription: {
          name: '',
          druglist: [],
          amount: 0,
          status: 0,
        },
        datafrom: {},
        itemdrugList: [],
        drugList: [],
        searchdrug: '',
        choices: [],
        dialogVisible: false,
        alldrugList: [],
        nondrugList: [],
        edit: 0,
        model: {},
        total2: 0,
        loading: false,
        isaside: false,
        asidewidth: "100%",
        total: 0,
        listQuery: {
          id: '',
          status: 1,
          name: '',
          scope: 0,
          ownId: this.$store.getters.id,
          aim: '',
          code: '',
          type: 1,
          pageSize: 20,
          pageNum: 1,
          isAdmin: '0'
        },
        modelList: [],
        queryModel: {
          name: '',
          range: '',
          type: ''
        }
      }
    },
    created() {
      this.getDataList()
    },
    methods: {
      removeList(index){ //删除行数
        this.dataList4.splice(index, 1)
      },
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/electronic_case_template/case/allTemplate'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.dataList1 = data.list
          } else {
            this.dataList1 = []
          }
          this.dataListLoading = false
        })
      },
      //获取复选框
      handleSelectionChange(val) {
        this.multipleSelection = val;
      },
      addTest() {
        this.dataList4=this.multipleSelection
        this.dataList3=this.multipleSelection
        this.dialogVisible = false
      },
      deldrug(row) {
        this.oneprescription.druglist = this.oneprescription.druglist.filter(item => {
          if (item.id === row.id)
            return false
          return true
        })
        this.oneprescription.amount = 0
        this.oneprescription.druglist.forEach(item => {
          this.oneprescription.amount += item.price
        })
        this.oneprescription.amount = Math.floor((this.oneprescription.amount + 0.5) * 100) / 100
      },
      choosedrug(val) {
        let flag = 1
        this.oneprescription.druglist.forEach(item => {
          if (item.id === val.id) {
            item.num += 1
            flag = 0
          }
        })
        if (flag) {
          this.oneprescription.amount += val.price
          this.oneprescription.amount = Math.floor((this.oneprescription.amount + 0.5) * 100) / 100
          this.oneprescription.druglist.push(val)
          this.oneprescription.druglist.forEach(item => {
            if (item.num === undefined)
              item.num = 1
          })
        }
      },
      deleteModel(id) {
        var ids = id ? [id] : this.multipleSelection.map(item => {
          return item.id
        })
        this.$confirm(`确定对这${ids.length}条数据进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/electronic_case_template/case/delete'),
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
      },
      createModel() {
        var datafrom=this.datafrom
        var ids = this.multipleSelection.map(item => {
          return item.icdId
        })
        this.$http({
          url: this.$http.adornUrl(`/electronic_case_template/case/save`),
          method: 'post',
          data: this.$http.adornData({'electronicCaseTemplate': datafrom, 'ids': ids},)
        }).then(({data}) => {
          this.confirmButtonDisabled = true
          if (data && data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })

        this.showaside()
      },
      updateModel(){
        var ids=this.dataList4.map(item => {
          return item.icdId
        })
        this.$http({
          url: this.$http.adornUrl(`/electronic_case_template/case/update`),
          method: 'put',
          data: this.$http.adornData({'electronicCaseTemplate':this.datafrom,'ids':ids},)
        }).then(({data}) => {
          this.confirmButtonDisabled = true
          if (data && data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
        this.showaside()
      },
      confirmItem() {
        this.nondrugList = this.alldrugList.filter(item => {
          if (this.choices.includes(item.id))
            return true
        })
        this.dialogVisible = !this.dialogVisible
      },
      addItem() {
        this.getIcdList()
        this.dialogVisible = true
      },
      getIcdList(){
          this.$http({
              url: this.$http.adornUrl('/icd/icd/list'),
              method: 'get',
              params: this.$http.adornParams({
                  'page': this.page.pageIndex,
                  'limit': this.page.pageSize,
                  'key': this.page.dataForm.key
              })
          }).then(({data}) => {
              if (data && data.code === 200) {
                  this.icdList = data.page.list
                  this.page.totalPage = data.page.totalCount
              } else {
                  this.icdList = []
                  this.page.totalPage = 0
              }
          })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.page.pageSize = val
        this.page.pageIndex = 1
        this.getIcdList()
        },
        // 当前页
      currentChangeHandle (val) {
        this.page.pageIndex = val
        this.getIcdList()
      },
      searchModel() {
        this.loading = true
        this.loading = false
      },
      showaside(type) {
        if (this.asidewidth === "100%")
          this.asidewidth = "0%"
        else
          this.asidewidth = "100%"
        this.isaside = !this.isaside
        if (type === 'edit')
          this.edit = 1
        else if (type === 'add') {
          this.choices = []
          this.nondrugList = []
          this.itemdrugList = []
          this.model = {}
          this.edit = 0
        } else
          this.edit = 0
      },
      editModel(tid){
        this.$http({
          url: this.$http.adornUrl(`/electronic_case_template/case/info/${tid}`),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.datafrom=data.case
            this.dataList4=data.icds
          }
        })
        this.itemdrugList = this.model.dmsMedicineModelItemList
        this.showaside('edit')
      }
    }
  }
</script>
<style>
  body .el-table th.gutter {
    display: table-cell !important;
  }

  .el-transfer-panel {
    width: 300px;
  }
</style>
