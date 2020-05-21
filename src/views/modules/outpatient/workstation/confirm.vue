<template>
  <div>
  <!-- 确诊 -->
  <el-container>
  <el-aside :width="mainwidth" style="background:white;">
    <el-tag style="margin-bottom:20px;margin-left:0px" type="info">初诊内容:</el-tag>
    <el-button style="float:right" @click="controlfast"><i v-show="!isclose" class="el-icon-caret-right" /><i v-show="isclose" class="el-icon-caret-left" />  快捷操作</el-button>
    <el-form :model="prerecord" disabled style="color:black">
      <el-form-item label="主诉"><el-input readonly v-model="prerecord.complain" type="textarea" :autosize="{ minRows: 2, maxRows: 3}" placeholder="主述" style="width:80%;float:right;"></el-input></el-form-item>
      <el-form-item label="现病史"><el-input readonly v-model="prerecord.patientSymptom" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="现病史"  style="width:80%;float:right;"></el-input></el-form-item>
      <el-form-item label="现病治疗情况"><el-input readonly v-model="prerecord.treatment" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="现病治疗情况" style="width:80%;float:right"></el-input></el-form-item>
      <el-form-item label="既往史"><el-input readonly v-model="prerecord.medicalHistory" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="既往史" style="width:80%;float:right"></el-input></el-form-item>
      <el-form-item label="过敏史"><el-input readonly v-model="prerecord.allergyHistory" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="过敏史" style="width:80%;float:right"></el-input></el-form-item>
      <el-form-item label="体格检查"><el-input readonly v-model="prerecord.healthCheckup" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="体格检查" style="width:80%;float:right"></el-input></el-form-item>
    </el-form>
    <el-tag style="margin-bottom:20px;margin-left:1px">检查检验结果:</el-tag>
    <el-form>
      <el-form-item label="检查结果"><el-input v-model="prerecord.checkResult" placeholder="检查结果" style="width:40%"></el-input></el-form-item>
      <el-form-item label="检验结果"><el-input v-model="prerecord.testResult" placeholder="检验结果" style="width:40%"></el-input></el-form-item>
    </el-form>
    <div style="margin-left:0px;margin-bottom:30px">
      <el-tag>评估诊断:</el-tag>
      <el-card style="width:85%">
        <el-button type="text" style="float:right" @click="addDis">添加诊断</el-button>
        <el-table :data="record">
          <el-table-column property="icdId" label="ID"></el-table-column>
          <el-table-column property="icdName" label="名称"></el-table-column>
          <el-table-column property="icdCode" label="编码"></el-table-column>
          <el-table-column width="80">
            <template slot-scope="scope">
              <el-button type="text" @click="deleteDis(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
    </div>
    <div>
      <el-button type="primary" style="float:right;margin-right:30px">提交</el-button>
    </div>
  </el-aside>
  <transition name="el-zoom-in-left">
  <el-main width="50%" v-show="isclose" style="border-style: dotted;border-width: 0px 0px 0px 1px;border-color:#C0C0C0;margin-top:-12px">
     <el-tabs v-model="activeName">
      <el-tab-pane label="常用诊断" name="first">
        <el-table :data="medicineDiseIdList" @row-click="selectDis">
          <el-table-column label="ICD编码" prop="icdCode"></el-table-column>
          <el-table-column label="名称" prop="icdName"></el-table-column>
        </el-table>
      </el-tab-pane>
     </el-tabs>
  </el-main>
  </transition>
  </el-container>
  <el-dialog title="诊断目录" :visible.sync="dialogTableVisible" top="50px">
    <div style="height:520px">
    <span>搜索诊断</span>
    <el-input style="width:200px" placeholder="搜索诊断" v-model="disQuery.name"></el-input>
    <el-table highlight-current-row @row-click="selectDis" :data="disList " style="margin-top:20px">
      <el-table-column property="icdId" label="ID"></el-table-column>
      <el-table-column property="icdName" label="名称"></el-table-column>
      <el-table-column property="icdCode" label="编码"></el-table-column>
    </el-table>
    <pagination style="margin-top:0px" v-show="total>0" :total="total" page-sizes="[]" :page.sync="disQuery.pageNum" :limit.sync="disQuery.pageSize" />
    </div>
  </el-dialog>
  </div>
</template>
<script>
import Pagination from '@/components/Pagination'
export default {
  props:['patient','registerId'],
  components: {Pagination},
  name:'Comfirm',
  data(){
    return{
      medicineDiseIdList:[],
      dialogTableVisible:false,
      total:0,
      record:[],
      electronicListList:[],
      disQuery: {
        catId: '',
        code: '',
        name: '',
        icd: '',
        status: '',
        pageSize: 8,
        pageNum: 1
      },
      disList:[],
      prerecord:{},
      activeName:'first',
      isclose:true,
      mainwidth:"60%",
      activeNames: ['1']
    };
  },
  created() {
    this.getmedicineDiseIdList()
    this.getIcd()
  },
  watch:{
    'patient' : function(newVal){
      this.patient = newVal
      this.getElectronicCase();
    },
  },
  methods:{
    deleteDis(row){
      this.record=this.record.filter(item=>{
        if(item.id===row.id)
          return false
        return true
      })
    },
    getIcd () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/icd/icd/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.icdPage.pageIndex,
          'limit': this.icdPage.pageSize,
          'key': this.icdPage.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.disList = data.page.list
          this.icdPage.totalPage = data.page.totalCount
        } else {
          this.icd = []
          this.icdPage.totalPage = 0
        }
        this.icdPage.dataListLoading = false
      })
    },
    getElectronicCase(){
      this.$http({
        url: this.$http.adornUrl(`/electronic_case/case/info/${this.registerId}`),
        method: 'get'
      }).then(({data}) => {
        this.confirmButtonDisabled = true
        if (data && data.code === 200) {
            this.prerecord = data.case.electronicCase
            this.record = data.case.icds
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    selectDis(val){
      this.$confirm('是否添加 '+val.name+' 到该患者?', '添加诊断', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          let flag = 1
          this.record.forEach(item=>{
            if(item.icd===val.icd){
              flag=0
            }
          })
          if(flag)
            this.record.push(val)
          else
            alert('已存在该诊断！')
          this.dialogTableVisible = false
        })
    },
    addDis(){
      this.dialogTableVisible=true
    },
    controlfast(){
      this.isclose=!this.isclose
      if(this.mainwidth==="60%")
        this.mainwidth="80%"
      else
        this.mainwidth="60%"
    },
    getmedicineDiseIdList(){
      this.$http({
        url: this.$http.adornUrl('/electronic_case/case/topFive'),
        method: 'get'
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.medicineDiseIdList = data.list
        } else {
          this.medicineDiseIdList = []
        }
      })
    }
  }
}
</script>
