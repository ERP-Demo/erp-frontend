<template>
  <!-- 检查申请 -->
  <el-container>
  <el-aside :width="mainwidth" style="background:white;padding:0 10px 0 0">
    <aside style="margin:0 0 0 0">
      <el-button type="text" size="medium" @click="addcheck"><i class="el-icon-circle-plus-outline" />新增项目</el-button>
      <el-button type="text" size="medium" @click="delcheck"><i class="el-icon-remove-outline" />删除项目</el-button>
      <el-button type="text" size="medium" @click="apply" ><i class="el-icon-circle-check" />开立项目</el-button>
      <el-button type="text" size="medium" @click="invalid"><i class="el-icon-circle-close" />作废项目</el-button>
      <el-button type="text" size="medium" @click="saveNonDrug"><i class="el-icon-upload2" />暂存</el-button>
      <el-button type="text" size="medium" @click="getNonDrug"><i class="el-icon-download" />取出暂存项</el-button>
      <el-button type="text" size="mini" @click="refresh"><i class="el-icon-refresh" />刷新</el-button>
      <el-button style="float:right" @click="controlfast"><i v-show="!isclose" class="el-icon-caret-right" /><i v-show="isclose" class="el-icon-caret-left" />  </el-button>
    </aside>
    <el-tag type="primary">项目金额总计:</el-tag>
    <el-tag type="warning">{{totalprice}} 元</el-tag>
    <el-table
    ref="multipleTable"
    :data="record"
    border
    tooltip-effect="dark"
    style="width: 100%"
    @selection-change="handleSelectionChange">
    <el-table-column
      align="center"
      type="selection"
      width="55"
      @selection-change="handleSelectionChange">
    </el-table-column>
    <el-table-column
    align="center"
      label="项目编码"
      width="120">
      <template slot-scope="scope">{{ scope.row.testSynthesizeId }}</template>
    </el-table-column>
    <el-table-column
    align="center"
      prop="testSynthesizeName"
      label="项目名称"
      width="200">
    </el-table-column>
    <el-table-column
    align="center"
      prop="testSynthesizePrice"
      label="单价"
      show-overflow-tooltip>
    </el-table-column>
    <el-table-column
    align="center"
      label="状态"
      show-overflow-tooltip>
    <template slot-scope="scope">
      <el-tag type="warning" v-if="scope.row.status===-1">未开立</el-tag>
      <el-tag type="danger" v-if="scope.row.status===0">已作废</el-tag>
      <el-tag type="info" v-if="scope.row.status===1">未缴费</el-tag>
      <el-tag type="info" v-if="scope.row.status===2">未登记</el-tag> 
      <el-tag type="info" v-if="scope.row.status===3">已登记</el-tag>
      <el-tag type="success" v-if="scope.row.status===4">已执行</el-tag>  
      <el-tag type="danger" v-if="scope.row.status===5">已退费</el-tag>  
    </template>
    </el-table-column>
    <el-table-column
    align="center"
      label=""
      show-overflow-tooltip>
      <template slot-scope="scope">
        <el-button type="text" v-if="scope.row.status===-1" @click="demand(scope.row)">检查要求</el-button>
        <el-button type="text" v-if="scope.row.status===4" @click="showresult(scope.row)">查看结果</el-button>
      </template>
    </el-table-column>
  </el-table>
  
  
  </el-aside>
  <transition name="el-zoom-in-left">
  <el-main width="50%" v-show="isclose" style="border-style: dotted;border-width: 0px 0px 0px 1px;border-color:#C0C0C0;margin-top:-12px">
     <el-tabs>
      <el-tab-pane label="检查模板" name="first">
        
        <el-table highlight-current-row @row-dblclick="addModel" @row-click="selectModel" stripe :data="自己加" height="230">
          <el-table-column label="模板名">
            <template slot-scope="scope">
              {{scope.row.name}}
            </template>
          </el-table-column>
          <el-table-column label="模板简介">
            <template slot-scope="scope">
              {{scope.row.aim}}
            </template>
          </el-table-column>
        </el-table>
        <el-card v-show="onemodel.name!==undefined" class="box-card" shadow="never" body-style="font-size: 14px;font-family:'微软雅黑';width:350px">
          <div slot="header" class="clearfix">
            <span>模板详情</span>
          </div>
          <p><b>模板名:</b> {{onemodel.name}}</p>
          <p><b>模板内容:</b></p>
          <p v-for="(nondrug,index) in onemodel.nondruglist" :key="index"><b></b> {{nondrug.name}}</p>
          <p><b>总金额：</b>{{onemodel.totalprice}} 元</p>
        </el-card>
     </el-tab-pane>
     <el-tab-pane label="常用检查项" name="second">
       <el-table :data="自己加" highlight-current-row  @row-click="addfreitem">
         <el-table-column label="项目名" prop="name"></el-table-column>
         <el-table-column label="项目单价" prop="price"></el-table-column>
         <el-table-column label="编码" prop="code"></el-table-column>
       </el-table>
     </el-tab-pane>
    </el-tabs>
  </el-main>
  </transition>
  <el-dialog title="检查目录" :visible.sync="dialogTableVisible" top="50px">
    <div style="height:450px">
      <el-form :inline="true" :model="listQuery" @keyup.enter.native="getDataList()">
        <el-form-item>
          <el-input v-model="listQuery.testSynthesizeName" placeholder="参数名" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="getDataList()">查询</el-button>
        </el-form-item>
      </el-form>
<!--    <el-table height="400px" @row-click="selectCheck" highlight-current-row :data="checkList.filter(data => !search || data.name.toLowerCase().includes(search.toLowerCase())||data.code.toLowerCase().includes(search.toLowerCase()))" style="margin-top:20px">-->
    <el-table height="400px" @row-click="selectCheck" highlight-current-row :data="checkList" style="margin-top:20px">
      <el-table-column property="testSynthesizeId" label="项目编码"></el-table-column>
      <el-table-column property="testSynthesizeName" label="项目名"></el-table-column>
      <el-table-column property="testSynthesizePrice" label="价格"></el-table-column>
    </el-table>
    </div>
  </el-dialog>
  <el-dialog title="填写检查要求" :visible.sync="demandVisible">
    <div style="height:260px">
      <el-form inline label-width="100px" >
        <el-form-item label="项目编码"><el-input disabled placeholder=""  v-model="dataForm.projectId"></el-input></el-form-item>
        <el-form-item label="项目名称"><el-input disabled placeholder=""  v-model="dataForm.projectName"></el-input></el-form-item>
        <el-form-item label="目的" ><el-input placeholder="" v-model="dataForm.purpose"></el-input></el-form-item>
        <el-form-item label="要求" ><el-input placeholder="" v-model="dataForm.requirements"></el-input></el-form-item>
        <el-form-item label="临床印象"><el-input placeholder=""  v-model="dataForm.clinicalImpression"></el-input></el-form-item>
        <el-form-item label="临床诊断" ><el-input placeholder="" v-model="dataForm.clinicalDiagnosis"></el-input></el-form-item>
        <el-form-item label="检查部位"><el-input placeholder=""  v-model="dataForm.checkThe"></el-input></el-form-item>
      </el-form>
      <el-button type="danger" style="float:right" @click="demandVisible=false">取消</el-button>
      <el-button type="primary" style="float:right;margin-right:10px" @click="submitDemand()">确认</el-button>
    </div>
  </el-dialog>
  <el-dialog title="检查结果" :visible.sync="resultvisible" top="10px"> 
    <div class="block" style="height:100%;width:100%">
      <span>检查结果：{{checkresult}}</span>
      <el-carousel trigger="click" height="500px" style="margin-top:30px">
        <el-carousel-item v-for="item in checkresultimg" :key="item">
           <img :src="item" style="height:100%">
        </el-carousel-item>
      </el-carousel>
    </div>
  </el-dialog>
  </el-container>
</template>
<script>
import {deepClone} from '@/utils'
export default {
  props:['patient'],
  name:'Inspection',
  data(){
    return{
      resultvisible:false,
      checkresult:'',
      checkresultimg:[],
      onemodel:{},
      totalprice:0.000,
      ref:[],
      dataForm: {},
      demandVisible:false,
      dialogTableVisible:false,
      isclose:true,
      checkList:[],
      mainwidth:"65%",
      search:'',
      listQuery: {
        testSynthesizeId:null,
        testSynthesizeName:null,
        format:null,
        testSynthesizePrice:null,
        expClassId: null,
        deptId: null,
        mnemonicCode:null,
        recordType: 1,
        createDate: null,
        status:1,
        pageSize:1000,
        pageNum:1,
        ids:'',
        name:'',
      },
      record:[]
    };
  },
   watch:{
    'patient' : function(newVal){
      this.patient = newVal
    },
  },
  activated () {
    this.getDataList()
  },
  methods:{
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/test_synthesize/synthesize/list'),
        method: 'get',
        params: this.$http.adornParams({
          'testSynthesizeName': this.listQuery.testSynthesizeName
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.checkList = data.page.list
        } else {
          this.checkList = []
        }
        this.dataListLoading = false
      })
    },
    showresult(row){
      this.checkresult = row.checkResult
      row.resultImgUrlList.split(',').forEach(item=>{
        this.checkresultimg.push(item)
      })
      this.resultvisible = true
    },
    addfreitem(val){
      this.selectCheck(val)
    },
    addModel(val){
      this.$confirm('是否确定将 模板:'+val.name+' 中的内容导入该患者的检查中', '导入模板', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
          val.nondruglist.forEach(item=>{
            item.status = -1
            let flag = 1
            this.record.forEach(com=>{
              if(com.id===item.id){
                flag = 0
              }
            })
            if(flag)
              this.record.push(item)
          })
        })
    },
    selectModel(val){
      this.onemodel = val
    },
    refresh(){
      this.$confirm('未开立的项目都将清除,确认刷新?', '刷新', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        })
    },
    handleSelectionChange(val){
      this.dataForm.projectName=this.checkList.map(item =>{return item.testSynthesizeName}).join(",")
      this.dataForm.projectId=this.checkList.map(item =>{return item.testSynthesizeId}).join(",")
      // this.dataForm.projectId=1
      this.ref = val
      this.totalprice = 0.00
      this.ref.forEach(item=>{
        this.totalprice+=item.price
      })
      this.totalprice = this.totalprice.toFixed(2)
    },
    submitDemand(){
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/requirements/requirements/save'),
        method: 'post',
        data: this.$http.adornData(this.dataForm)
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
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
      this.demandVisible = false
    },
    demand(row){
      this.dataForm.projectId=row.testSynthesizeId
      this.dataForm.projectName=row.testSynthesizeName
      this.demandVisible = true
      this.check = deepClone(row)
    },
    selectCheckred(val){
      let flag = 1
      val.status = -1
      this.record.forEach(item=>{
        if(item.id===val.id){
          flag=0
        }
      })
      if(flag)
        this.record.push(val)
      this.dialogTableVisible = false
    },
    selectCheck(val){
      this.$confirm('是否添加 '+val.testSynthesizeName+' 到该患者?', '添加检查', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          let flag = 1
          val.status = -1
          this.record.forEach(item=>{
            if(item.id===val.id){
              flag=0
            }
          })
          if(flag)
            this.record.push(val)
          else
            alert('已存在该检查项目！')
          this.dialogTableVisible = false
        })
    },
    addcheck(){
      this.dialogTableVisible = true
    },
    delcheck(){
      this.record = this.record.filter(item=>{
        if(item.status===-1)
          if(this.ref.includes(item))
            return false
        return true
      })
    },
    controlfast(){
      this.isclose=!this.isclose
      if(this.mainwidth==="65%")
        this.mainwidth="80%"
      else
        this.mainwidth="65%"
    }
  },
}
</script>

