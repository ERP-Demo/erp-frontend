<template>
  <!-- 成药处方 -->
  <div>
    <el-container>
      <el-aside :width="mainwidth" style="background:white;padding:0 10px 0 0">
        <aside style="margin:0 0 0 0">
          <el-button type="text" size="medium" @click="addpre"><i class="el-icon-circle-plus-outline"/>新增处方
          </el-button>
          <el-button type="text" size="medium" @click="deletepre"><i class="el-icon-remove-outline"/>删除处方
          </el-button>
          <el-button type="text" size="medium" @click="dataFormSubmit"><i class="el-icon-circle-check"/>开立处方
          </el-button>
          <el-button type="text" size="medium" @click="toVoid"><i class="el-icon-circle-close"/>作废处方
          </el-button>
          <el-button type="text" size="medium"><i class="el-icon-upload2"/>暂存
          </el-button>
          <el-button type="text" size="medium"><i class="el-icon-download"/>取出暂存项
          </el-button>
          <el-button type="text" size="medium" @click="refresh"><i class="el-icon-refresh"/>刷新</el-button>
          <el-button style="float:right" @click="controlfast"><i v-show="!isclose" class="el-icon-caret-right"/>
            <i v-show="isclose" class="el-icon-caret-left"/></el-button>
        </aside>
        <el-table
                ref="multipleTable"
                :data="inData"
                tooltip-effect="dark"
                style="width: 100%"
                :cell-style="style"
                :header-cell-style="style"
                @selection-change="handleSelectionChange">
          <el-table-column
                  align="center"
                  type="selection"
                  width="55">
          </el-table-column>
          <el-table-column
                  align="center"
                  label="处方名称">
            <template slot-scope="scope">{{ scope.row.prescriptionName }}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  prop="amount"
                  label="总金额(元)">
          </el-table-column>
          <el-table-column label="状态">
            <template slot-scope="scope">
              <el-tag type="success" v-if="scope.row.status===-1">未开立</el-tag>
              <el-tag type="success" v-if="scope.row.status===0">已开立</el-tag>
              <el-tag type="success" v-if="scope.row.status===1">已作废</el-tag>
            </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label=""
                  show-overflow-tooltip>
            <template slot-scope="scope">
              <el-button type="text" v-if="scope.row.status===-1" @click="showDetail(scope.row)">编辑</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-aside>
      <transition name="el-zoom-in-left">
        <el-main width="50%" v-show="isclose" style="border-style: dotted;border-width: 0px 0px 0px 1px;border-color:#C0C0C0;margin-top:-12px">
          <el-tabs v-model="activeName">
            <el-tab-pane label="成药模板" name="first">
              <el-table :data="models" height="230" @row-click="selectmodel" @row-dblclick="addmodel">
                <el-table-column label="模板名">
                  <template slot-scope="scope">
                    {{scope.row.drugModelName}}
                  </template>
                </el-table-column>
                <el-table-column label="目的">
                  <template slot-scope="scope">
                    {{scope.row.drugModelIntroduction}}
                  </template>
                </el-table-column>
              </el-table>
              <el-card v-show="model.drugModelName!==undefined" class="box-card" shadow="never"
                       body-style="font-size: 14px;font-family:'微软雅黑';width:350px">
                <div slot="header" class="clearfix">
                  <span>{{model.drugModelName}}</span>
                </div>
                <p><b>模板目的：</b>{{model.drugModelIntroduction}}</p>
                <p><b>模板总金额: </b>{{model.amount}}</p>
                <p><b>模板项目：</b></p>
                <p v-for="(drug,index) in model.druglist" :key="index"><b></b> {{drug.name}}</p>
              </el-card>
            </el-tab-pane>
          </el-tabs>
        </el-main>
      </transition>
    </el-container>
    <el-dialog title="处方详细" :visible.sync="dialogTableVisible" width="1500px" top="20px">
      <el-container>
        <el-aside width="30%" style="padding:0 0 0 0;margin:0 0 0 0">
          <div>
            <el-tag type="primary" style="width:100%">常用药品</el-tag>
            <el-table :data="freqlist" height="200px" @row-click="choosedrug" >
              <el-table-column label="药品名" prop="name">
              </el-table-column>
              <el-table-column label="价格(元)" prop="price" width="100px"></el-table-column>
            </el-table>
          </div>
          <el-tag type="primary" style="width:100%;height:30px">药品目录

            <el-button type="primary" size="mini" style="width: 20px;;float:right">
              <svg-icon icon-class="search" style="margin-left:-6px;"/>
            </el-button>
            <el-input size="mini" placeholder="药品名称" v-model="searchdrug" style="width:60%;float:right"
                      @change="getdrugList"></el-input>
          </el-tag>

          <el-table :data="drugList" height="300px" @row-click="choosedrug">
            <el-table-column label="药品名" prop="drugsName"></el-table-column>
            <el-table-column label="价格(元)" prop="drugsPrice" width="100px"></el-table-column>
          </el-table>
          <el-pagination
                  @size-change="sizeChangeHandle"
                  @current-change="currentChangeHandle"
                  :current-page="drugsPage.pageIndex"
                  :page-sizes="[10, 20, 50, 100]"
                  :page-size="drugsPage.pageSize"
                  :total="drugsPage.totalPage"
                  layout="total, sizes, prev, pager, next, jumper">
          </el-pagination>
        </el-aside>
        <el-main>
          <el-tag type="primary">项目金额总计:</el-tag>
          <el-tag type="warning">{{oneprescription.amount}}元</el-tag>
          <el-button type="primary" style="float:right" @click="createpre" v-if="!edit">增加处方</el-button>
          <el-button type="primary" style="float:right" @click="changepre" v-if="edit">修改处方</el-button>
          <el-input style="width:200px;margin-right:20px;float:right"
                    v-model="oneprescription.prescriptionName" placeholder="处方名"></el-input>
          <el-table height="500px" :data="oneprescription.druglist" :cell-style="style"
                    :header-cell-style="style">
            <el-table-column property="drugsId" label="药品id" width="150"></el-table-column>
            <el-table-column property="drugsName" label="药品名" width="150"></el-table-column>
            <el-table-column property="drugsNorms" label="规格" width="200"></el-table-column>
            <el-table-column property="drugsPrice" label="单价"></el-table-column>
            <el-table-column property="pharmacyNum" label="库存"></el-table-column>
            <el-table-column label="数量" width="130px">
              <template slot-scope="scope">
                <el-input-number controls-position="right" style="width:100px" :min="1" :max="scope.row.pharmacyNum"
                                 size="mini" @change="changenum(scope.row)"
                                 v-model="scope.row.drugsNum"></el-input-number>
              </template>
            </el-table-column>
            <el-table-column label="使用方法" width="130">
              <template slot-scope="scope">
                <el-input placeholder="用法" v-model="scope.row.drugsUse"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="天数" width="100px">
              <template slot-scope="scope">
                <el-input v-model="scope.row.drugsDay" placeholder=""></el-input>
              </template>
            </el-table-column>
            <el-table-column label="用量" width="100px">
              <template slot-scope="scope">
                <el-input v-model="scope.row.drugsUsenum" placeholder=""></el-input>
              </template>
            </el-table-column>
            <el-table-column width="50px">
              <template slot-scope="scope">
                <el-button type="text" @click="deldrug(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-main>
      </el-container>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    props: ['patient','registerId'],
    name: 'Prescription',
    data() {
      return {
        modelvisivle: false,
        model: {},
        models: [],
        freqlist: [],
        drugs:[],
        refs: [],
        prescriptionList: [],
        prescriptionDetails:[],
        oneprescription: {
          prescriptionName: '',
          druglist: [],
          amount: 0,
          status: -1
        },
        edit: false,
        test: '',
        num: 0,
        dialogTableVisible: false,
        activeName: 'first',
        isclose: true,
        record: {
          main: 'test'
        },
        page: {
          pageNum: 1,
          pageSize: 10,
        },
        searchdrug: '',
        drugList: [],
        drugsPage: {
          dataForm: {
            key: ''
          },
          pageIndex: 1,
          pageSize: 10,
          totalPage: 0,
        },
        inData: [],
        data: {
          prescriptionName: '',
          detailed: []
        },
        total: 0,
        mainwidth: "65%",
        rowId: 1
      };
    },
    watch: {
      'patient': function (newVal) {
        this.patient = newVal
        this.getOpenList()
        this.getDataList2()
      },
    },
    created() {
      this.listModel()
      this.getDataList1()
    },
    methods: {
      getOpenList(){
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/prescription/prescription/queryByrId/'+this.registerId),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.inData= data.list
            this.inData.forEach(item => {
              item.amount = item.prescriptionPrice
              item.status=item.prescriptionState
            })
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },

      //成药模板的显示
      getDataList1 () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/drug_model/model/list'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.models = data.page.list
            console.log(data.page.list)
          } else {
            this.models = []
          }
          this.dataListLoading = false
        })

      },
      addmodel(val) {
        val.amount = Math.floor((val.amount) * 100) / 100
        val.status = -1
        val.prescriptionName=val.drugModelName
        this.$http({
          url: this.$http.adornUrl(`/drug_model/model/info/${val.drugModelId}`),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 200) {
            val.amount=data.price
            this.oneprescription.druglist=data.list
            console.log(this.inData)
            this.inData.push(val)
          }
        })
       // this.inData.push(val)
      },
      selectmodel() {
        this.model.amount = Math.floor((this.model.amount) * 100) / 100
        this.modelvisivle = true
      },
      listModel() {
        let data = {}
        data.scope = 0
        data.ownId = this.$store.getters.id
        data.type = 1
        data.pageSize = 1000
        data.pageNum = 1
        data.isAdmin = 0
      },
      addfreitem(val) {
        this.selectCheck(val)
      },
      refresh() {
        this.$confirm('未开立的处方都将清除,确认刷新?', '刷新', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        })
      },
      deletepre() {
        let data = this.refs[0].name
        this.prescriptionList = this.prescriptionList.filter(item => {
          if (item.name === data)
            return false
          return true
        })
      },
      handleSelectionChange(val) {
        this.refs = val
      },
      createpre() {
        this.edit = false
        var data = {
          prescriptionName: this.oneprescription.prescriptionName,
          druglist: [],
          status: -1,
          amount: 0,
          rowId: this.rowId
        }
        this.rowId++
        this.oneprescription.druglist.map(item => {
          data.druglist.push({
            drugsId: item.drugsId,
            drugsNum: item.drugsNum,
            drugsPrice: item.drugsPrice,
            drugsUse: item.drugsUse,
            drugsDay: item.drugsDay,
            drugsUsenum: item.drugsUsenum
          })
          data.amount += item.drugsPrice
        })
        this.inData.push(data)
        this.prescriptionList.push(data)
        this.dialogTableVisible = false
      },
      changepre() {
        this.dialogTableVisible = false
      },
      changenum() {
        this.oneprescription.amount = 0
        this.oneprescription.druglist.forEach(item => {
          this.oneprescription.amount += item.drugsPrice * item.num
        })
        this.oneprescription.amount = Math.floor((this.oneprescription.amount) * 100) / 100
      },
      deldrug(row) {
        this.oneprescription.druglist = this.oneprescription.druglist.filter(item => {
          if (item.drugsId === row.drugsId)
            return false
          return true
        })
      },
      choosedrug(val) {
        let flag = 1
        this.oneprescription.druglist.forEach(item => {
          if (item.drugsId === val.drugsId) {
            item.num += 1
            flag = 0
          }
        })
        if (flag) {
          this.oneprescription.amount += val.drugsPrice
          this.oneprescription.amount = Math.floor((this.oneprescription.amount) * 100) / 100
          this.oneprescription.druglist.push(val)
          this.oneprescription.druglist.forEach(item => {
            if (item.num === undefined)
              item.num = 1
          })

        }
      },
      async getdrugList() {
        let data = {}
        data.pageSize = this.page.pageSize
        data.pageNum = this.page.pageNum
        data.status = 1
        data.typeId = 101
        data.name = this.searchdrug
      },
      addpre() {
        this.getDrugs()
        this.dialogTableVisible = true
      },
      showDetail(row) {
        this.edit = true
        this.oneprescription = row
        this.dialogTableVisible = true
      },
      controlfast() {
        this.isclose = !this.isclose
        if (this.mainwidth === "65%")
          this.mainwidth = "80%"
        else
          this.mainwidth = "65%"
      },
      getDrugs() {
        this.$http({
          url: this.$http.adornUrl('/drugs/detailed/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.drugsPage.pageIndex,
            'limit': this.drugsPage.pageSize,
            'name': this.drugsPage.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.drugList = data.list
            this.drugsPage.totalPage = data.page.totalCount
          } else {
            this.drugList = []
            this.drugsPage.totalPage = 0
          }
        })
      },
      // 每页数
      sizeChangeHandle(val) {
        this.drugsPage.pageSize = val
        this.drugsPage.pageIndex = 1
        this.getDrugs()
      },
      // 当前页
      currentChangeHandle(val) {
        this.drugsPage.pageIndex = val
        this.getDrugs()
      },
      dataFormSubmit() {
        console.log({'data': this.refs, 'registrationId': this.patient.registrationId});
        this.$http({
              url: this.$http.adornUrl('/prescription/prescription/addDrugsAndDetailed'),
              method: 'post',
              data: this.$http.adornData(this.refs, false)
            }).then(({data}) => {
              this.confirmButtonDisabled = true
              if (data && data.code === 200) {
                var ids=[]
                this.refs.map(item => {
                  ids.push(item.rowId)
                })
            this.inData.map(item => {
              if(ids.indexOf(item.rowId) > -1) {
                item.status = 0
                this.$set(item, 'preId', data.map[item.rowId])
              }
            })
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      toVoid() {
        var ids = this.refs.map(item => {
          return item.preId
        })
        this.$http({
          url: this.$http.adornUrl('/prescription/prescription/toVoid'),
          method: 'post',
          data: ids
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.$message({
              message: '作废成功',
              type: 'success',
              duration: 1000
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      style(){
        return 'text-align: center'
      }
    }
  }
</script>