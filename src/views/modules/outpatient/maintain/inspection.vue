<template>
    <!-- 化验
    模板管理 -->
  <div class="div1">
    <el-container>
    <!--  -->
    <transition name="el-zoom-in-left">

    <el-aside :width="asidewidth" style="margin-top:0;background:white;padding: 0 0 0 0">
          <aside style="height:50px;margin:0 0 0 0">
            <el-tag size="large">化验模板</el-tag>

          </aside>
        <div style="padding: 0 10px 0 10px">
        <el-card v-if="!isaside">
          <el-form style="background:white" :model="listQuery" inline>
            <el-form-item label="模板名称:" style="width:280px">
              <el-input v-model="listQuery.name" placeholder="模板名称"></el-input>
            </el-form-item>
            <el-form-item label="范围:" style="width:280px">
            <el-select placeholder="请选择范围" v-model="listQuery.scope" clearable style="width: 130px" class="filter-item">
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
        <el-table v-loading="loading" :data="modelList" stripe border highlight-current-row>
          <el-table-column align="center" label="模板编号" prop="id" width="80">
            <template slot-scope="scope">
              {{scope.row.id}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="模板名称" prop="name">
            <template slot-scope="scope">
              {{scope.row.name}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="模板简介" prop="aim">
            <template slot-scope="scope">
              {{scope.row.aim}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="模版编码" prop="code">
            <template slot-scope="scope">
              {{scope.row.code}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="范围" prop="scope" width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.scope===0">个人</span>
              <span v-if="scope.row.scope===1">科室</span>
              <span v-if="scope.row.scope===2">全院</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="模板类型" prop="type"  width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.type===0">检查</span>
              <span v-if="scope.row.type===1">检验</span>
              <span v-if="scope.row.type===2">处置</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" prop="id" width="150px">
            <template slot-scope="scope"> 
              <el-button type="primary" size="mini" @click="editModel(scope.row)">修改</el-button>
              <el-button type="danger" size="mini" @click="deleteModel(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
         <pagination v-show="total>0" :total="total" :page.sync="listQuery.pageNum" :limit.sync="listQuery.pageSize" />
        </div>
      </div>
    </el-aside>
    </transition>
    <!-- 编辑模板 -->
    <el-main style="padding:0 0 0 0;"  v-if="isaside">
      <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="edit" @click="updateModel">提交修改</el-button>
      <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="!edit" @click="createModel">新建模板</el-button>
      <el-button type="danger" @click="showaside">取消</el-button>
      <el-form :model="model" label-width="140px" inline>
        <el-form-item label="模板名称">
          <el-input placeholder="请输入模板名称" v-model="model.name" style="width:300px"></el-input>
        </el-form-item>
        <el-form-item label="模板简介">
          <el-input placeholder="模板简介" v-model="model.aim" style="width:300px"></el-input>
        </el-form-item>
        <el-form-item v-if='edit' label="模板编码">
          <el-input placeholder="模板编码" v-model="model.code" disabled style="width:300px"></el-input>
        </el-form-item>
        <el-form-item v-if='edit'  label="创建时间">
          <el-input placeholder="" v-model="model.createTime"  style="width:300px" disabled></el-input>
        </el-form-item>
        <el-form-item label="范围:" style="width:280px">
          <el-select placeholder="请选择范围" v-model="listQuery.scope" clearable style="width: 130px" class="filter-item">
            <el-option v-for="item in [{key:0,value:'个人'},{key:1,value:'科室'},{key:2,value:'全院'}]" :key="item.key" :label="item.value" :value="item.key" />
          </el-select>
        </el-form-item>
      </el-form>
      <el-button type="primary" style="margin-bottom:10px;margin-left:10px" @click="addItem">编辑项目</el-button>

        <el-table style="margin-bottom:50px" :data="dataList">
          <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
          <el-table-column   label="化验项目名" prop="testModelName" width="330px"></el-table-column>
          <el-table-column label="项目简介" prop="testModelIntroduction" width="200px"></el-table-column>
<!--        <el-table-column label="药品单价(元)" prop="drugModelPrice" width="70px"></el-table-column>-->
<!--        <el-table-column label="药品数量" prop="drugModelNum" width="70px"></el-table-column>-->
<!--        <el-table-column label="总价" prop="drugModelPrice" width="70px"></el-table-column>-->
<!--        <el-table-column label="拼音码" prop="drugModelEnglishcode" width="200px"></el-table-column>-->
<!--        <el-table-column label="使用建议" prop="drugModelProposal"></el-table-column>-->
<!--        <el-table-column label="频次" prop="drugModelFrequency">-->
      </el-table>
    </el-main>
    </el-container>
    <el-dialog title="增加化验项目" :visible.sync="dialogVisible" width="700px"  top="10px">
      <el-container>
        <el-aside width="100%" style="padding:0 0 0 0;margin:0 0 0 0">
          <el-tag type="primary" style="width:100%;height:30px">化验项目目录
            <el-button type="primary" size="mini"  style="width: 20px;;float:right"><svg-icon icon-class="search" style="margin-left:-6px"/></el-button>
            <el-input  size="mini" placeholder="化验项目名称" style="width:60%;float:right" @change="getdrugList(0)"></el-input>
          </el-tag>
          <el-table :data="drugList" height="500px" @row-click="choosedrug">
            <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
            <el-table-column label="化验项目id" prop="id"></el-table-column>
            <el-table-column label="化验项目名" prop="name"></el-table-column>
          </el-table>
          <pagination layout="prev, pager, next" auto-scroll="false" style="margin-top:0px" page-sizes="[]"  v-show="total2>0" :total="total2" :page.sync="page.pageNum" :limit.sync="page.pageSize" @pagination="getdrugList(0)" />
          <div>
          </div>
        </el-aside>
      </el-container>
      <span slot="footer" class="dialog-footer">
      <el-button type="primary" style="float:right" @click="addDrug">确定</el-button>
    </span>
    </el-dialog>
  </div>
</template>
<script>
import Pagination from '@/components/Pagination'

export default {
  components: {Pagination},
  data(){
    return{
      dataList:[],
      page:{
        pageNum:1,
        pageSize:10,
      },
      oneprescription:{
        name:'',
        druglist:[],
        amount:0,
        status:0,
      },
      itemdrugList:[],
      drugList:[],
      dialogVisible:false,
      edit:0,
      model:{},
      total2:0,
      isaside:false,
      asidewidth:"100%",
      total:0,
      listQuery:{
        id:'',
        status:1,
        name:'',
        scope:0,
        ownId:this.$store.getters.id,
        aim:'',
        code:'',
        type:1,
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
  activated () {
    this.getDataList()
  },
  methods:{
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/test_model/model/list'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.dataList = data.page.list
        } else {
          this.dataList = []
        }
        this.dataListLoading = false
      })

    },
    addDrug(){
      this.dialogVisible = false
      this.itemdrugList = this.oneprescription.druglist
      this.itemdrugList.forEach(item=>{
        item.totalprice = Math.floor((item.price*item.num)*100)/100
      })
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
        this.oneprescription.amount = Math.floor((this.oneprescription.amount)*100)/100
        this.oneprescription.druglist.push(val)
        this.oneprescription.druglist.forEach(item=>{
          if(item.num===undefined)
            item.num=1
        })
      }
    },
    addItem(){
      this.dialogVisible = true
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
        this.itemdrugList = []
        this.model = {}
        this.edit = 0
      }
      else
        this.edit = 0
    },
    async editModel(){
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
