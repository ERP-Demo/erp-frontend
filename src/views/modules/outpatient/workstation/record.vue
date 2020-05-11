<template>
  <div>
    <!-- 病历首页 -->
    <el-container class="el-container">
      <el-aside :width="mainwidth" style="background:white;">
        <el-tag style="margin-bottom:20px">病史内容:</el-tag>
        <el-button type="text" style="margin-left:30px" @click="saveCasePage"><i class="el-icon-upload2" />暂存</el-button>
        <el-button type="text" style="margin-left:30px" @click="getCasePage"><i class="el-icon-download" />取出暂存病历</el-button>
        <el-button style="float:right" @click="controlfast"><i v-show="!isclose" class="el-icon-caret-right" /><i v-show="isclose" class="el-icon-caret-left" />  快捷操作</el-button>
        <el-form :model="record" label-width="100px">
          <el-form-item label="主诉"><el-input v-model="priliminaryDise.complain" type="textarea" :autosize="{ minRows: 2, maxRows: 3}" placeholder="主述" style="width:80%"></el-input></el-form-item>
          <el-form-item label="现病史"><el-input v-model="priliminaryDise.patientSymptom" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="现病史" style="width:80%" ></el-input></el-form-item>
          <el-form-item label="现病治疗情况"><el-input v-model="priliminaryDise.treatment" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="现病治疗情况" style="width:80%"></el-input></el-form-item>
          <el-form-item label="既往史"><el-input v-model="priliminaryDise.medicalHistory" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="既往史" style="width:80%"></el-input></el-form-item>
          <el-form-item label="过敏史"><el-input v-model="priliminaryDise.allergyHistory" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="过敏史" style="width:80%"></el-input></el-form-item>
          <el-form-item label="体格检查"><el-input v-model="priliminaryDise.healthCheckup" type="textarea" :autosize="{ minRows: 1, maxRows: 3}" placeholder="体格检查" style="width:80%"></el-input></el-form-item>
          <el-form-item label="发病时间">
            <el-date-picker
                    v-model="priliminaryDise.onsetTime"
                    align="right"
                    type="date"
                    placeholder="选择日期"
                    :picker-options="pickerOptions">
            </el-date-picker>
          </el-form-item>
        </el-form>
        <el-tag style="margin-bottom: 15px;">评估诊断:</el-tag>
        <el-card style="width:85%">
          <el-button type="text" style="float:right" @click="addIcd()">添加诊断</el-button>
          <el-table :data="icdZd">
            <el-table-column label="ID" prop="icdId"></el-table-column>
            <el-table-column label="名称" prop="icdName"></el-table-column>
            <el-table-column label="编码" prop="icdCode" ></el-table-column>
            <el-table-column width="80">
              <template slot-scope="scope">
                <el-button type="text" @click="deleteZd(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
        <div style="float:right;width:25%;margin-top:40px;margin-right: 20px;">
          <el-button type="primary" @click="submitPriliminaryDise">提交初诊</el-button>
        </div>
      </el-aside>
      <transition name="el-zoom-in-left">
        <el-main width="50%" v-show="isclose" style="border-style: dotted;border-width: 0px 0px 0px 1px;border-color:#C0C0C0;margin-top:-12px">
          <el-tabs v-model="activeName" @tab-click="handleClick">
            <el-tab-pane label="病历模板" name="first">
              <el-table stripe :data="models" height="230" @row-click="selectmodel" @row-dblclick="addmodel">
                <el-table-column label="病历名">
                  <template slot-scope="scope">
                    {{scope.row.name}}
                  </template>
                </el-table-column>
                <el-table-column label="主要诊断">
                  <template slot-scope="scope">
                    {{scope.row.dis}}
                  </template>
                </el-table-column>
              </el-table>
              <el-card  v-if="model.name!==undefined" class="box-card" shadow="never" body-style="font-size: 14px;font-family:'微软雅黑';width:350px">
                <div slot="header" class="clearfix">
                  <span>{{model.name}}</span>
                </div>
                <p><b>主述：</b>{{model.chiefComplaint}}</p>
                <p><b>现病史：</b>{{model.historyOfPresentIllness}}</p>
                <p><b>现病治疗情况: </b>{{model.historyOfTreatment}}</p>
                <p><b>既往史：</b>{{model.pastHistory}}</p>
                <p><b>过敏史：</b>{{model.allergies}}</p>
                <p><b>体格检查: </b>{{model.healthCheckup}}</p>
                <p><b>主要诊断: </b>{{model.dis}}</p>
              </el-card>
            </el-tab-pane>
            <el-tab-pane label="常用诊断" name="second">
              <el-table :data="medicineDiseIdList" @row-click="selectDis">
                <el-table-column label="ICD编码" prop="icd"></el-table-column>
                <el-table-column label="名称" prop="name"></el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="历史病历" name="third">
              <el-table highlight-current-row height="230" :data="history" @row-click="selecthistory">
                <el-table-column label="主要诊断" prop="priliminaryDiseStrList">
                </el-table-column>
                <el-table-column label="就诊时间" prop="createTime">
                </el-table-column>
              </el-table>
              <el-card v-show="historyitem.priliminaryDiseStrList!==undefined" class="box-card" shadow="never" body-style="font-size: 14px;font-family:'微软雅黑';width:470px;height:500px">
                <div slot="header" class="clearfix">
                  <span>病历详情</span>
                </div>
                <el-scrollbar style="height:100%">
                  <div style="width:100%">
                    <p><b>就诊时间: </b>{{historyitem.createTime}}</p>
                    <p><b>主要诊断: </b>{{historyitem.priliminaryDiseStrList}}<p>
                    <p><b>主述: </b>{{historyitem.chiefComplaint}}</p>
                    <p><b>现病史: </b>{{historyitem.historyOfPresentIllness}}</p>
                    <p><b>现病治疗情况: </b>{{historyitem.historyOfTreatment}}</p>
                    <p><b>既往史: </b>{{historyitem.pastHistory}}</p>
                    <p><b>过敏史: </b>{{historyitem.allergies}}</p>
                    <p><b>体格检查: </b>{{historyitem.healthCheckup}}</p>
                    <p><b>发病时间: </b>{{historyitem.startDate}}</p>
                    <p><b>检查结果: </b>{{historyitem.checkResult}}</p>
                    <p><b>检验结果: </b>{{historyitem.testResult}}</p>
                    <p><b>成药处方: </b>{{historyitem.medicinePrescriptionStrList}}</p>
                    <p><b>草药处方: </b>{{historyitem.herbalPrescriptionStrList}}</p>
                    <p><b>检验项目: </b>{{historyitem.testStrList}}</p>
                    <p><b>检查项目: </b>{{historyitem.checkStrList}}</p>
                    <p><b>处置项目: </b>{{historyitem.dispositionStrList}}</p>
                  </div>
                </el-scrollbar>
              </el-card>
            </el-tab-pane>
          </el-tabs>
        </el-main>
      </transition>
    </el-container>
    <el-dialog title="诊断目录" :visible.sync="dialogTableVisible" top="50px">
      <div style="height:520px">
        <span>搜索诊断</span>
        <el-input style="width:200px" placeholder="搜索诊断" v-model="disQuery.name" @change="getIcd"></el-input>
        <el-table highlight-current-row @row-click="selectDis" :data="icd " style="margin-top:20px;width: 800px;margin-left: 40px" >
          <el-table-column property="icdId" label="ID"></el-table-column>
          <el-table-column property="icdName" label="名称"></el-table-column>
          <el-table-column property="icdCode" label="编码"></el-table-column>
        </el-table>
        <el-pagination
                @size-change="sizeChangeHandle"
                @current-change="currentChangeHandle"
                :current-page="icdPage.pageIndex"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="icdPage.pageSize"
                :total="icdPage.totalPage"
                layout="total, sizes, prev, pager, next, jumper"
                style="text-align: center;height: 60px"></el-pagination>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import {getDmsDislist,parseList} from '@/api/diagnosis'
  import {submitPriliminaryDise,selectEndCaseHistoryByReg} from '@/api/outpatient/dmscase'
  import {getAllStaffModel} from '@/api/outpatient/dmscasemodel'
  import Pagination from '@/components/Pagination'
  import {selectByType,addfre,delfre} from '@/api/outpatient/frequentuse'
  import {parseTime,deepClone} from '@/utils'
  import {saveCasePage,getCasePage} from '@/api/outpatient/save'
  export default {
    props:['patient','registerId'],
    name:'Record',
    components: {Pagination},
    data(){
      return{
        datetable:[],
        history:[],
        historyitem:{},
        pickerOptions: {
          disabledDate(time) {
            return time.getTime() > Date.now();
          },
          shortcuts: [{
            text: '今天',
            onClick(picker) {
              picker.$emit('pick', new Date());
            }
          }, {
            text: '昨天',
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24);
              picker.$emit('pick', date);
            }
          }, {
            text: '一周前',
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', date);
            }
          }]
        },
        dialogTableVisible:false,
        activeName:'first',
        isclose:false,
        record:{},
        medicineDiseIdList:[],//常用诊断
        priliminaryDise:{
          complain:'',//主述
          patientSymptom:'',//现病史
          treatment:'',//现治疗情况
          medicalHistory:'',//既往史
          allergyHistory:'',//过敏史
          healthCheckup:'',//体格检查
          registrationId:'',//
          patientId:'',
          registerId:'',
          priliminaryDiseStrList:'',
          priliminaryDiseIdList:'',
          onsetTime:'',//发病时间
          name:'',
          gender:'',
          ageStr:'',
          icdId:'',
          icdName:'',
          icdCode:''
        },
        mainwidth:"80%",
        activeNames: ['1'],
        data2:[
          {
            date: '0001',
            name: '王小虎1',
            address: '38岁'
          },
          {
            date: '0002',
            name: '王小虎2',
            address: '39岁'
          }
        ],
        total:0,
        disQuery: {
          catId: '',
          code: '',
          name: '',
          icd: '',
          status: '',
          pageSize: 8,
          pageNum: 1
        },
        models:[],
        model:{},
        disList:[],
        icd:[],
        icdZd:[],
        icdPage:{
          dataForm: {
            key: ''
          },
          pageIndex: 1,
          pageSize: 10,
          totalPage: 0,
          dataListLoading: false,
          dataListSelections: [],
          addOrUpdateVisible: false
        }
      };
    },
    created(){
      this.getAllStaffModel()
    },
    watch:{
      'patient' : function(newVal, oldVal){
        this.patient = newVal
        this.selectEndCaseHistoryByReg()
        this.getElectronicCase()
      },
    },
    methods:{
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
            this.icd = data.page.list
            this.icdPage.totalPage = data.page.totalCount
          } else {
            this.icd = []
            this.icdPage.totalPage = 0
          }
          this.icdPage.dataListLoading = false
        })
      },
      sizeChangeHandle (val) {
        this.icdPage.pageSize = val
        this.icdPage.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.icdPage.pageIndex = val
        this.getDataList()
      },
      addmodel(val){
        this.$confirm('是否加载病历模板 '+val.name+' ?', '加载病历模板', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          this.priliminaryDise = deepClone(val)
          let diss = ''
          this.priliminaryDise.priliminaryDiseIdList.forEach(item=>{
            diss += (item+',')
          })
          diss = diss.substr(0,diss.length-1)
          parseList(diss).then(res=>{
            this.record = res.data
          })
        })
      },
      selectmodel(val){
        this.model = val
      },
      getAllStaffModel(){
        getAllStaffModel(this.$store.getters.id).then(res=>{
          this.models = res.data.staffList
          this.models.forEach(item=>{
            item.dis = ''
            item.priliminaryDiseStrList.forEach(dis=>{
              item.dis+=(dis+',')
            })
            item.dis = item.dis.substr(0,item.dis.length-1)
          })
        })
      },
      saveCasePage(){
        let data  =this.priliminaryDise
        data.registrationId = this.patient.registrationId
        this.priliminaryDise.patientId=this.patient.patientId
        this.$http({
          url: this.$http.adornUrl(`/electronic_case/case/saveRidis`),
          method: 'post',
          data: this.$http.adornData(this.priliminaryDise)
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
      },
      getCasePage(){
        let data  =this.priliminaryDise
        data.registrationId = this.patient.registrationId
        this.priliminaryDise.patientId=this.patient.patientId
        this.$http({
          url: this.$http.adornUrl(`/electronic_case/case/getRidis`),
          method: 'post',
          data: this.$http.adornData(this.priliminaryDise)
        }).then(({data}) => {
          this.datetable=data.list
          if(this.priliminaryDise.complain==null){
            this.datetable=data.list
          }else if(this.datetable!=null){
            this.priliminaryDise=data.list
          }else {
            this.datetable=data.list
          }
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
      },
      selecthistory(val){
        this.historyitem = val
      },
      selectEndCaseHistoryByReg(){
        selectEndCaseHistoryByReg(this.patient.registrationId).then(res=>{
          this.history = res.data.dmsCaseHistoryList
          this.history.forEach(item=>{
            item.createTime = parseTime(item.createTime)
            item.startDate = parseTime(item.startDate)
          })
        })
      },
      getElectronicCase(){
        this.$http({
          url: this.$http.adornUrl(`/electronic_case/case/info/${this.registerId}`),
          method: 'get'
        }).then(({data}) => {
          this.confirmButtonDisabled = true
          if (data && data.code === 200) {
            if (data.case) {
              this.icdZd = data.case.icds
              this.priliminaryDise = data.case.electronicCase
            }
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      submitPriliminaryDise(){
        // this.$refs['record'].validate((valid) => {
        //   if (valid) {
        this.priliminaryDise.patientId=this.patient.patientId
        this.priliminaryDise.registerId=this.registerId
        let ids=this.icdZd.map(item => {
          return item.icdId
        })
        this.$http({
          url: this.$http.adornUrl(`/electronic_case/case/${!this.record.id ? 'save' : 'update'}`),
          method: !this.record.id ? 'post' : 'put',
          data: this.$http.adornData({'electronicCase':this.priliminaryDise,'icdId':ids})
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
            this.record={brand_right:0};
          } else {
            this.$message.error(data.msg)
          }
        })
        //   }
        // })
      },
      deleteZd(row){
        this.icdZd=this.icdZd.filter(item=>{
          if(item.icdId===row.icdId)
            return false
          return true
        })
      },
      selectDis(val){
        this.$confirm('是否添加 '+val.icdName+' 到该患者?', '添加诊断', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          let flag = 1
          this.icdZd.forEach(item=>{
            if(item.icdId===val.icdId){
              flag=0
            }
          })
          if(flag)
            this.icdZd.push(val)
          else
            alert('已存在该诊断！')
          this.dialogTableVisible = false
        })
      },
      async getDis(){
        const res = await getDmsDislist(this.disQuery)
        this.disList = res.data.list
        this.total = res.data.total
      },
      addIcd(){
        this.dialogTableVisible=true
      },
      controlfast(){
        this.isclose=!this.isclose
        if(this.mainwidth==="60%")
          this.mainwidth="80%"
        else
          this.mainwidth="60%"
      }
    }
  }
</script>
<style>
  .el-scrollbar__wrap {
    overflow-x: hidden;
  }
</style>
