<template>
  <!-- 门诊医生工作台 -->
  <div class="div1">
    <el-container class="el-container">
    <!-- 左侧患者列表 -->
    <transition name="el-zoom-in-left">
    <el-aside v-show="isAside" style=" width:350px;padding:0" class="aside">
      <div>
      <el-card style="width:350px">
      <span style="font-family: 微软雅黑;margin-right: 4px;">选择患者：</span>
      <el-input v-model="search" placeholder="患者名" style="width: 180px;" class="filter-item input1" />
      <el-button style="margin-left:10px" type="primary"  circle icon="el-icon-refresh" @click="getPatientList"></el-button>
      <el-tabs type="card" v-model="activeName" style="margin-top:15px">
      <el-tab-pane  label="本人" name="first">
      <el-tag style="width:100%">待诊患者</el-tag>
      <el-table @row-click="handleCurrentChange" highlight-current-row stripe :data="personalWaitList === null ? personalWaitList : personalWaitList.filter(data => !search || data.patientName.toLowerCase().includes(search.toLowerCase())||data.patientMedicalRecordNo.toLowerCase().includes(search.toLowerCase()))" height="255">
        <el-table-column align="center" label="名字" width="90px">
          <template slot-scope="scope">
            {{scope.row.patient.patientName}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="年龄" width="65px">
          <template slot-scope="scope">
            {{scope.row.patient.patientAge}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="操作" width="150px">
          <template slot-scope="scope">
            <el-button type="text" @click.stop="removePatien(scope.row)">解绑</el-button>
          </template>

        </el-table-column>
      </el-table>
      <el-tag type="success" style="width:100%">诊中患者</el-tag>
      <el-table @row-click="continuing" highlight-current-row stripe :data="personalDuringList === null ? personalDuringList:personalDuringList.filter(data => !search || data.patient.patientName.toLowerCase().includes(search.toLowerCase())||data.registerId.toLowerCase().includes(search.toLowerCase()))" height="255">
        <el-table-column align="center" label="病历号" width="150px">
          <template slot-scope="scope">
            {{scope.row.registerId}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="名字" width="90px">
          <template slot-scope="scope">
            {{scope.row.patient.patientName}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="年龄" width="65px">
          <template slot-scope="scope">
            {{scope.row.patient.patientAge}}
          </template>
        </el-table-column>
      </el-table>
      <el-tag type="warning" style="width:100%">已诊患者</el-tag>
      <el-table stripe :data="personalEndList === null ? personalEndList:personalEndList.filter(data => !search || data.patient.patientName.toLowerCase().includes(search.toLowerCase())||data.registerId.toLowerCase().includes(search.toLowerCase()))" height="255">
        <el-table-column align="center" label="病历号" width="150px">
          <template slot-scope="scope">
            {{scope.row.registerId}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="名字" width="90px">
          <template slot-scope="scope">
            {{scope.row.patient.patientName}}
          </template>
        </el-table-column>
        <el-table-column align="center" label="年龄" width="65px">
          <template slot-scope="scope">
            {{scope.row.patient.patientAge}}
          </template>
        </el-table-column>
      </el-table>
      </el-tab-pane>
      <el-tab-pane label="科室" name="second">
        <el-tag style="width:100%">待诊患者</el-tag>
        <el-table v-loading="loadDepp" @row-click="bindPatient" highlight-current-row stripe :data="deptWaitList === null ? deptWaitList:deptWaitList.filter(data => !search || data.patient.patientName.toLowerCase().includes(search.toLowerCase())||data.registerId.toLowerCase().includes(search.toLowerCase()))" height="820">
          <el-table-column align="center" label="病历号" width="150px">
            <template slot-scope="scope">
              {{scope.row.registerId}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="名字" width="90px">
            <template slot-scope="scope">
              {{scope.row.patient.patientName}}
            </template>
          </el-table-column>
          <el-table-column align="center" label="年龄" width="65px">
            <template slot-scope="scope">
              {{scope.row.patient.patientAge}}
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      </el-tabs>
      </el-card>
      </div>
    </el-aside>
    </transition>
    <!-- 医生工作台 -->
    <el-main class="box-main">
      <div class='popContainer' v-if="mask"></div>
      <div style="border-style: dotted;border-width: 0px 0px 1px 0px;border-color:#C0C0C0;padding: 0 0 10px 0;margin-bottom:-10px">
        <el-button size="mini" circle type="primary" @click="showAside"><i v-show="isAside" class="el-icon-arrow-left" /><i v-show="!isAside" class="el-icon-arrow-right" /></el-button>
        <span style="margin-left:15px;font-size:14px;font-family: '微软雅黑';">当前病人：</span>
        <el-tag class="painfo" color="white" style="width:15%;font-size:15px">姓名: <span style="olor:black;font-size: 14px;font-family:'微软雅黑';">{{patient.patientName}}</span></el-tag>
        <el-tag class="painfo" color="white" style="width:20%;font-size:15px">就诊号: <span style="color:black;font-size: 14px;font-family:'微软雅黑';">{{registerId}}</span></el-tag>
        <el-tag class="painfo" color="white" style="width:10%;font-size:15px">性别: <span style="color:black;font-size: 14px;font-family:'微软雅黑';">{{patient.patientSex}}</span></el-tag>
        <el-tag class="painfo" color="white" style="width:10%;font-size:15px">年龄: <span style="color:black;font-size: 14px;font-family:'微软雅黑';">{{patient.patientAge}}</span></el-tag>
        <el-button type="text" style="margin-left:30px;letter-spacing: 5px;" @click="endDiagnosis"><i class="el-icon-success" />诊毕</el-button>
      </div>
    <div>
      <el-tabs v-model="activeName2" type="card" style="margin-top:20px">
        <el-tab-pane label="病历首页" name="first"><Record ref="record" v-bind:patient="patient" v-bind:registerId="registerId"></Record></el-tab-pane>
        <el-tab-pane label="检查申请" name="second" @click="AllList()"><Inspection ref="inspection" v-bind:patient="patient" v-bind:registerId="registerId"></Inspection></el-tab-pane>
        <el-tab-pane label="门诊确诊" name="fourth"><Comfirm ref="comfirm" v-bind:patient="patient" v-bind:registerId="registerId" ></Comfirm></el-tab-pane>
        <el-tab-pane label="成药处方" name="fiveth"><Prescription ref="prescription" v-bind:patient="patient"></Prescription></el-tab-pane>
        <el-tab-pane label="处置申请" name="sixth"><Handle v-bind:patient="patient" v-bind:registerId="registerId"></Handle></el-tab-pane>
        <el-tab-pane label="患者账单" name="seventh"><Bill ref="bill" v-bind:patient="patient"></Bill></el-tab-pane>
      </el-tabs>
    </div>
    </el-main>
    </el-container>
  </div>
</template>

<script>
import Record from '@/views/modules/outpatient/workstation/record'
import Inspection from '@/views/modules/outpatient/workstation/inspection'
import Prescription from '@/views/modules/outpatient/workstation/prescription'
import Handle from '@/views/modules/outpatient/workstation/handle'
import Comfirm from '@/views/modules/outpatient/workstation/confirm'
import Bill from '@/views/modules/outpatient/workstation/bill'
  export default{
    components: {Record,Inspection,Prescription,Handle,Comfirm,Bill},
    data(){
      return{
        mask:true,
        loadDepp:true,
        patient:{},
        deptWaitList:[],
        personalDuringList:[],
        personalEndList:[],
        personalWaitList:[],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        dialog1: false,
        dialog2: false,
        search:'',
        isAside: true,
        activeName: 'first',
        activeName2: 'first',
        registerId: '',
      };
    },
    created(){
      this.getPatientList()
    },
    methods: {
      endDiagnosis(){
          this.showAside()
          this.getPatientList()
      },
      AllList(){
        alert("1111")
      },
      continuing(val){
          this.$confirm('确认继续诊断患者 '+val.patient.patientName+'?', '就诊', {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'success'
          }).then(()=>{
              this.showAside()
              this.patient = val.patient
              this.registerId=val.registerId
              this.$refs.record.controlfast()
            })

      },
      bindPatient(val){
          this.$confirm('确认绑定患者 '+val.patient.patientName+'?', '就诊', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
            this.$http({
              url: this.$http.adornUrl('/activiti/consultation'),
              method: 'post',
              params: this.$http.adornParams({'processInstanceId':val.processInstanceId},false)
            }).then(({data}) => {
              if (data && data.code === 200) {
                this.$message.success("成功绑定该患者")
                this.getPatientList()
              } else {
                this.$message.error(data.msg)
              }
            })
        })
      },
      async handleCurrentChange(val){
        this.$confirm('确认开始诊断患者 '+val.patient.patientName+'?', '就诊', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          this.$http({
            url: this.$http.adornUrl('/activiti/startDiagnosis'),
            method: 'post',
            params: this.$http.adornParams({'processInstanceId':val.processInstanceId},false)
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.$message.success("开始诊断")
              this.patient = val
              this.getPatientList()
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      async getPatientList(){
        this.loadDepp = true
        this.$http({
          url: this.$http.adornUrl('/register/register/refreshPatient'),
          method: 'get'
        }).then(res=>{
          this.deptWaitList = res.data.deptWaitList
          this.personalDuringList = res.data.personalDuringList
          // this.personalEndList = res.data.personalEndList
          this.deptWaitList = res.data.deptWaitList
          this.personalWaitList = res.data.personalWaitList
          console.log(this.personalWaitList);
          this.loadDepp = false
        }).catch(err=>{
          this.loadDepp = false
          this.$message.error(err)
        })
      },
      showAside(){
        if(this.isAside===false){
          this.isAside=true;
          this.mask = true
        }
        else{
          this.isAside = false
          this.mask = false
        }
      },
      removePatien(val){
        this.$confirm('确认解除绑定患者 '+val.patient.patientName+'?', '解除绑定', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          this.$http({
            url: this.$http.adornUrl('/activiti/unconsultation'),
            method: 'post',
            params: this.$http.adornParams({'processInstanceId':val.processInstanceId},false)
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.$message.success("成功解除绑定该患者")
              this.getPatientList()
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
  }
}
</script>
<style>
  div.popContainer {
    position: absolute;
    top: 20px;
    left: 400px;
    right: 35px;
    bottom: 35px;
    z-index: 100;
    background: rgba(0,0,0,0.3);
    border-radius: 4px;
  }

  .div1{
    font-family:  "微软雅黑";
  }
  .card1{
    font-size: 14px;
    font-family: "微软雅黑";
    line-height: 20px;
  }
  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
  }
  .input1{
    width:200px;
  }
  .box-card {
    width: 480px;
  }
  ::-webkit-scrollbar {
    width: 8px;
    height: 8px;
    background-color: #fff;
  }
  ::-webkit-scrollbar-thumb {
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
    background-color:#F0F8FF
  }
</style>


