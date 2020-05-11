<template>
    <!-- 成药模板管理 -->
  <div class="div1">
    <el-container>
    <!--  -->
    <transition name="el-zoom-in-left">

    <el-aside :width="asidewidth" style="margin-top:0;background:white;padding: 0 0 0 0">
          <aside style="height:50px;margin:0 0 0 0">
            <el-tag size="large">药品模板</el-tag>
            
          </aside>
        <div style="padding: 0 10px 0 10px">
        <el-card v-if="!isaside">
          <el-form style="background:white" :model="listQuery" inline>
            <el-form-item label="模板名称:" style="width:280px">
              <el-input v-model="listQuery.drugModelName" placeholder="模板名称"></el-input>
            </el-form-item>
            <el-form-item label="范围:" style="width:280px">
              <el-select placeholder="请选择范围" v-model="listQuery.drugModelRange" clearable style="width: 130px" class="filter-item">
               <el-option v-for="item in [{key:0,value:'个人'},{key:1,value:'科室'},{key:2,value:'全院'}]" :key="item.key" :label="item.value" :value="item.key" />
              </el-select>
            </el-form-item>
            <el-form-item style="width:400px">
              <el-button type="primary" @click="searchModel" v-if="!isaside">搜索模板</el-button>
              <el-button type="primary" @click="showaside('add')" v-if="!isaside">新建模板</el-button>
            </el-form-item>
          </el-form>
        </el-card>
        <div style="background:white;">
        <el-table v-loading="loading" :data="datalist3" stripe border highlight-current-row>
          <el-table-column align="center" label="模板编号" prop="drugModelId" width="80">
            <template slot-scope="scope">
              {{scope.row.drugModelId}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="模板名称" prop="drugModelName">
            <template slot-scope="scope">
              {{scope.row.drugModelName}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="模板简介" prop="drugModelIntroduction">
            <template slot-scope="scope">
              {{scope.row.drugModelIntroduction}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="模版编码" prop="drugModelCode">
            <template slot-scope="scope">
              {{scope.row.drugModelCode}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="范围" prop="drugModelRange" width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.drugModelRange===0">个人</span>
              <span v-if="scope.row.drugModelRange===1">科室</span>
              <span v-if="scope.row.drugModelRange===2">全院</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="模板类型" prop="drugModelType"  width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.drugModelType===1">成药模板</span>
              <span v-if="scope.row.drugModelType===2">草药模板</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" prop="id" width="150px">
            <template slot-scope="scope"> 
              <el-button type="primary" size="mini"  @click="showaside('edit'),editModel(scope.row.drugModelId)">修改</el-button>
              <el-button type="danger" size="mini" @click="deleteModel(scope.row.drugModelId)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
         <pagination v-show="total>0" :total="total" :page.sync="listQuery.pageNum" :limit.sync="listQuery.pageSize" @pagination="getModelList" />
        </div>
      </div>
    </el-aside>
    </transition>
    <!-- 修改模板 -->
    <el-main style="padding:0 0 0 0;"  v-if="isaside">
      <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="edit" @click="updateModel">提交修改</el-button>
      <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="!edit" @click="createModel">新建模板</el-button>
      <el-button type="danger" @click="showaside">取消</el-button>
      <el-form :model="datafrom" label-width="140px" inline>
        <el-form-item label="模板名称">
          <el-input placeholder="请输入模板名称" v-model="datafrom.drugModelName  " style="width:250px"></el-input>
        </el-form-item>
        <el-form-item label="模板简介">
          <el-input placeholder="模板简介" v-model="datafrom.drugModelIntroduction" style="width:250px"></el-input>
        </el-form-item>
<!--          v-if='edit'-->
<!--          disabled-->
        <el-form-item label="模板编码" >
          <el-input placeholder="模板编码"  v-model="datafrom.drugModelCode" style="width:250px"></el-input>
        </el-form-item>
        <el-form-item label="模板范围">
          <el-select placeholder="请选择范围" v-model="datafrom.drugModelRange" clearable style="width: 250px" class="filter-item">
               <el-option v-for="item in [{key:0,value:'个人'},{key:1,value:'科室'},{key:2,value:'全院'}]" :key="item.key" :label="item.value" :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item   label="模板类型">
            <el-select placeholder="请选择类型" v-model="datafrom.drugModelType" clearable style="width: 250px" class="filter-item">
                <el-option v-for="item in [{key:1,value:'成药模板'},{key:2,value:'草药模板'}]" :key="item.key" :label="item.value" :value="item.key" />
            </el-select>
        </el-form-item>
      </el-form>
      <el-button type="primary" style="margin-bottom:10px;margin-left:10px" @click="addItem">编辑项目</el-button>
      <el-table style="margin-bottom:50px" :data="dataList1">
        <el-table-column prop="drugsName"  label="药品名" width="330px"></el-table-column>
        <el-table-column label="药品规格" prop="drugsNorms" width="200px"></el-table-column>
        <el-table-column label="药品单价(元)" prop="drugsPrice" width="70px"></el-table-column>
        <el-table-column label="用法" prop="drugsUsage" width="70px"></el-table-column>
        <el-table-column label="用量" prop="drugsDosage" width="70px"></el-table-column>
        <el-table-column label="禁忌" prop="drugsTaboo" width="200px"></el-table-column>
        <el-table-column label="生产厂商" prop="drugsProducer"></el-table-column>
        <el-table-column label="批准文号" prop="drugsApprovalNumber">
        </el-table-column>
      </el-table>

    </el-main>
    </el-container>
    <el-dialog title="增加药品" :visible.sync="dialogVisible" width="1500px" top="10px">
        <el-container>
        <el-aside width="30%" style="padding:0 0 0 0;margin:0 0 0 0">
            <el-tag type="primary" style="width:100%;height:40px;line-height: 40px;margin-top: 2%">药品目录
            </el-tag>
            <pagination layout="prev, pager, next" auto-scroll="false" style="margin-top:0px" page-sizes="[]"  v-show="total2>0" :total="total2" :page.sync="page.pageNum" :limit.sync="page.pageSize" @pagination="getdrugList(0)" />
          <div>
          </div>
        </el-aside>
        <el-main>
            <el-button type="primary" style="float:right;margin-top: -6%;margin-right: 6%" @click="addDrug">确定</el-button>
          <el-table :data="dataList" cell-style="text-align:center" header-cell-style="text-align:center" style="margin-bottom: 5%" @selection-change="handleSelectionChange">
<!--              <el-table-column width="50px">-->
<!--                <template slot-scope="scope">-->
<!--                  <el-button type="text" @click="deldrug(scope.row)">删除</el-button>-->
<!--                </template>-->
<!--              </el-table-column>-->
              <el-table-column
                  type="selection"
                  header-align="center"
                  align="center"
                  width="50">
              </el-table-column>
            <el-table-column property="drugsName" label="药品名" width="150"></el-table-column>
            <el-table-column property="drugsNorms" label="规格" width="200"></el-table-column>
            <el-table-column property="drugsPrice" label="单价"></el-table-column>
<!--            <el-table-column label="数量" width="130px"> -->
<!--              <template slot-scope="scope">-->
<!--                <el-input-number controls-position="right" style="width:100px" :min="1" :max="100" size="mini" @change="changenum(scope.row)" v-model="scope.row.num"></el-input-number>-->
<!--              </template>-->
<!--            </el-table-column>-->
            <el-table-column label="用法" width="130px" prop="drugsUsage">
            </el-table-column>
            <el-table-column label="用量" width="130" prop="drugsDosage">
<!--              <template slot-scope="scope">-->
<!--                <el-input placeholder="用法" v-model="scope.row.medicalAdvice"></el-input>-->
<!--              </template>  -->
            </el-table-column>
            <el-table-column label="禁忌" width="180px" prop="drugsTaboo">
<!--          <template slot-scope="scope">-->
<!--            <el-select v-model="scope.row.frequency" placeholder="" style="width:120px">-->
<!--              <el-option  v-for="item in [{key:1,label:'一天一次'},{key:2,label:'一天三次'}]" :key="item.key" :label="item.label" :value="item.key" ></el-option>-->
<!--            </el-select>-->
<!--          </template>-->
            </el-table-column>
            <el-table-column label="生产厂商" width="180px" prop="drugsProducer">
    <!--          <template slot-scope="scope">-->
    <!--           <el-input v-model="scope.row.days" placeholder=""></el-input>-->
    <!--          </template>-->
            </el-table-column>
            <el-table-column label="批准文号" width="180px" prop="drugsApprovalNumber">
    <!--          <template slot-scope="scope">-->
    <!--           <el-input v-model="scope.row.usageNum" placeholder=""></el-input>-->
    <!--          </template>-->
            </el-table-column>
<!--        <el-table-column label="单位" width="130px">-->
<!--          <template slot-scope="scope">-->
<!--            <el-select v-model="scope.row.usageNumUnit" placeholder="" style="width:120px">-->
<!--              <el-option  v-for="item in [{key:1,label:'片'},{key:2,label:'支'},{key:3,label:'瓶'},{key:2,label:'克'}]" :key="item.key" :label="item.label" :value="item.key" ></el-option>-->
<!--            </el-select>-->
<!--          </template>-->
<!--        </el-table-column>-->
          </el-table>
        </el-main>
        </el-container>
      </el-dialog>
  </div>
</template>
<script>
import {listModel,updateModel,createModel,deleteModel} from '@/api/drugmodel'
import {getdrugList,selectById} from '@/api/drug'
import Pagination from '@/components/Pagination'
import { deepClone,parseTime } from '@/utils'

export default {
    components: {Pagination},
  data(){
    return{
      dataList:[],
      dataList1:[],
      datalist3:[],
      multipleSelection:[],
      page:{
        pageNum:1,
        pageSize:10,
      },
        ides:'',
      oneprescription:{
        name:'',
        druglist:[],
        amount:0,
        status:0,
      },
      itemdrugList:[],
      drugList:[],
      searchdrug:'',
      choices:[],
      dialogVisible:false,
      alldrugList:[],
      nondrugList:[],
      edit:0,
      model:{},
      total2:0,
      loading:false,
      isaside:false,
      asidewidth:"100%",
      total:0,
      datafrom:{
          drugModelName:'',
          drugModelIntroduction:'',
          drugModelCode:'',
          drugModelRange:'',
          drugModelType:'',
          //drugsId:[],
      },
      listQuery:{
        drugModelId:'',
        status:1,
        drugModelName:"",
        drugModelRange:0,
        ownId:this.$store.getters.id,
        drugModelIntroduction:'',
        drugModelCode:'',
        drugModelType:1,
        pageSize:20,
        pageNum:1,
        isAdmin:'0'
      },
      modelList:[],
      queryModel:{
        name:'',
        range:'',
        type:''
      }
    }
  },
  created(){
    this.getModelList()
  },
  activated () {
    this.getDataList1()
    this.getDataList2()
  },
  methods:{
    //成药模板的显示
      getDataList1 () {
          this.dataListLoading = true
          this.$http({
              url: this.$http.adornUrl('/drug_model/model/list'),
              method: 'get',
              //params: this.$http.adornParams()
          }).then(({data}) => {
              if (data && data.code === 200) {
                  this.datalist3 = data.page.list
                  //console.log("数据"+data.list)
              } else {
                  this.dataList3 = []
              }
              this.dataListLoading = false
          })
      },
    // 获取数据列表
    getDataList2 () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/drugs/reports/all'),
        method: 'get',
        //params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList = data.list
          //console.log("数据"+data.list)
        } else {
          this.dataList = []
        }
        this.dataListLoading = false
      })
    },
      //获取复选框
      handleSelectionChange(val) {
          this.multipleSelection = val;
      },
    addDrug(drugsId){
        console.log("11111111111111111111111111")
      this.dialogVisible = false
      //var ids=this.multipleSelection.map(item =>{return item.drugsId}).join(",")
        var ids = this.drugsId ? [drugsId] : this.multipleSelection.map(item => {
            return item.drugsId
        })
        console.log("id:"+ids)

        this.$http({
            url: this.$http.adornUrl('/drugs/reports/selectByIds/'+ids),
            method: 'get',
            data: this.$http.adornData()
            //params: this.$http.adornParams()
        }).then(({data}) => {
            if (data && data.code === 200) {
                this.dataList1 = data.list
                console.log("数据"+data.list)
            } else {
                this.dataList1 = []
            }

        })
      //this.itemdrugList = this.dataList
      // this.itemdrugList.forEach(item=>{
      //   item.totalprice = Math.floor((item.price*item.num)*100)/100
      // })
    },
    async getdrugList(type) {
      let data = {}
      data.pageSize = this.page.pageSize
      data.pageNum = this.page.pageNum
      if(type!==0)
        data.typeId = type
      data.name = this.searchdrug
      const response = await getdrugList(data)
      this.drugList = response.data.list
      this.total2 = response.data.total
    },
    deldrug(row){
      this.oneprescription.druglist = this.oneprescription.druglist.filter(item=>{
        if(item.id===row.id)
          return false
        return true
      })
      this.oneprescription.amount = 0
      this.oneprescription.druglist.forEach(item=>{
        this.oneprescription.amount += item.price
      })
      this.oneprescription.amount = Math.floor((this.oneprescription.amount+0.5)*100)/100
    },
    choosedrug(val){
      let flag = 1
      this.oneprescription.druglist.forEach(item=>{
        if(item.id===val.id){
          item.num+=1
          flag=0
        }
      })
      if(flag){
        this.oneprescription.amount +=val.price
        this.oneprescription.amount = Math.floor((this.oneprescription.amount+0.5)*100)/100
        this.oneprescription.druglist.push(val)
        this.oneprescription.druglist.forEach(item=>{
          if(item.num===undefined)
            item.num=1
        })
      }
    },
    deleteModel(id){
    var ids = id ? [id] : this.multipleSelection.map(item => {
        return item.id
    })
    this.$confirm(`确定对这${ids.length}条数据进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
    }).then(() => {
        this.$http({
            url: this.$http.adornUrl('/drug_model/model/delete'),
            method: 'delete',
            data: this.$http.adornData(ids, false)
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
    })
    },
    createModel(){
        var ids=this.multipleSelection.map(item => {
            return item.drugsId
        })
        this.$http({
            url: this.$http.adornUrl(`/drug_model/model/save`),
            method: 'post',
            data: this.$http.adornData({'drugModel':this.datafrom,'ids':ids},)
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
                        this.getDataList1()
                    }
                })
            } else {
                this.$message.error(data.msg)
            }
        })
      this.showaside()
    },
    updateModel(){
      let data = deepClone(this.model)
      data.nonDrugIdList = deepClone(this.choices)
      data.createTime = ''
      data.ownId = this.$store.getters.id
      updateModel(data).then(res=>{
        this.$notify({
          title: '成功',
          message: '修改成功',
          type: 'success',
          duration: 2000
        })
        this.getModelList()
      })
      this.showaside()
    },
    confirmItem(){
      this.nondrugList = this.alldrugList.filter(item=>{
        if(this.choices.includes(item.id))
          return true
      })
      this.dialogVisible=!this.dialogVisible
    },
    addItem(){
      this.dialogVisible = true
      this.getdrugList()
    },
    getAllNondrug(){
      getAllNondrug().then(res=>{
        this.alldrugList = res.data
        this.alldrugList.forEach(item=>{
          item.label = item.name
          item.key = item.id
        })
      })
    },
    async getModelList(){
      listModel(this.listQuery).then(res=>{
        this.modelList = res.data.list
        this.modelList.forEach(model=>{
          model.dmsMedicineModelItemList.forEach(item=>{
          selectById(item.id).then(res=>{
            item.name = res.data.name
            item.format = res.data.format
            item.price = res.data.price
            item.totalprice = item.price*item.num
            item.mnemonicCode = res.data.mnemonicCode
            })
          })
          model.createTime = parseTime(model.createTime)
        })
        this.total = res.data.total
      })
      
    },
    searchModel(){
      this.loading = true
      this.getModelList()
      this.loading = false
    },
    showaside(type){
      if(this.asidewidth==="100%")
        this.asidewidth="0%"
      else
        this.asidewidth="100%"
      this.isaside=!this.isaside
      if(type==='edit')
        this.edit = 1
      else if(type==='add'){
        this.choices = []
        this.nondrugList = []
        this.itemdrugList = []
        this.model = {}
        this.edit = 0 
      }
      else
        this.edit = 0
    },
    async editModel(row){
      this.model = deepClone(row)
      this.itemdrugList = this.model.dmsMedicineModelItemList
      this.showaside('edit')
    }
  }
}
</script>
<style>
  body .el-table th.gutter{
display: table-cell!important;
}
.el-transfer-panel{
    width: 300px;
}
</style>
