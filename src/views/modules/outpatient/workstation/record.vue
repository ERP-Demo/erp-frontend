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
          <el-tabs>
            <el-tab-pane label="病历模板" name="first">
              <el-table stripe height="230" @row-click="selectmodel" @row-dblclick="addmodel">
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
              <el-table @row-click="selectDis">
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
        <el-input style="width:200px" placeholder="搜索诊断" @change="getIcd"></el-input>
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
  import {deepClone} from '@/utils'
  export default {
    props: ['patient', 'registerId'],
    name: 'Record',
    data() {
      return {
        datetable: [],
        history: [],
        historyitem: {},
        pickerOptions: {
          disabledDate(time) {
            return time.getTime() > Date.now();
          },
          dialogTableVisible: false,
          isclose: false,
          record: {},
          priliminaryDise: {
            complain: '',//主述
            patientSymptom: '',//现病史
            treatment: '',//现治疗情况
            medicalHistory: '',//既往史
            allergyHistory: '',//过敏史
            healthCheckup: '',//体格检查
            registrationId: '',//
            patientId: '',
            registerId: '',
            priliminaryDiseStrList: '',
            priliminaryDiseIdList: '',
            onsetTime: '',//发病时间
            name: '',
            gender: '',
            ageStr: '',
            icdId: '',
            icdName: '',
            icdCode: ''
          },
          mainwidth: "80%",
          model: {},
          icd: [],
          icdZd: [],
          icdPage: {
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
        },
        watch: {
          'patient': function (newVal) {
            this.patient = newVal
            this.getCasePage()
          },
        },
        methods: {
          getIcd() {
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
          sizeChangeHandle(val) {
            this.icdPage.pageSize = val
            this.icdPage.pageIndex = 1
            this.getDataList()
          },
          // 当前页
          currentChangeHandle(val) {
            this.icdPage.pageIndex = val
            this.getDataList()
          },
          addmodel(val) {
            this.$confirm('是否加载病历模板 ' + val.name + ' ?', '加载病历模板', {
              confirmButtonText: '确认',
              cancelButtonText: '取消',
              type: 'success'
            }).then(() => {
              this.priliminaryDise = deepClone(val)
            })
          },
          selectmodel(val) {
            this.model = val
          },
          saveCasePage() {
            let data = this.priliminaryDise
            data.registrationId = this.patient.registrationId
            this.priliminaryDise.patientId = this.patient.patientId
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
          getCasePage() {
            let data = this.priliminaryDise
            data.registrationId = this.patient.registrationId
            this.priliminaryDise.patientId = this.patient.patientId
            this.$http({
              url: this.$http.adornUrl(`/electronic_case/case/getRidis`),
              method: 'post',
              data: this.$http.adornData(this.priliminaryDise)
            }).then(({data}) => {
              this.datetable = data.list
              if (this.priliminaryDise.complain == null) {
                this.datetable = data.list
              } else if (this.datetable != null) {
                this.priliminaryDise = data.list
              } else {
                this.datetable = data.list
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
          selecthistory(val) {
            this.historyitem = val
          },
          submitPriliminaryDise() {
            this.priliminaryDise.patientId = this.patient.patientId
            this.priliminaryDise.registerId = this.registerId
            this.icdZd = this.icdZd.filter(item => {
              this.priliminaryDise.icdId = item.icdId
              this.priliminaryDise.icdName = item.icdName
              this.priliminaryDise.icdCode = item.icdCode
            })
            this.$http({
              url: this.$http.adornUrl(`/electronic_case/case/${!this.record.id ? 'save' : 'update'}`),
              method: !this.record.id ? 'post' : 'put',
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
                this.record = {brand_right: 0};
              } else {
                this.$message.error(data.msg)
              }
            })
            //   }
            // })
            // this.priliminaryDise.registrationId = this.patient.registrationId
            // this.record.forEach(item=>{
            //   this.priliminaryDise.priliminaryDiseStrList+=(item.name+',')
            //   this.priliminaryDise.priliminaryDiseIdList+=(item.id+',')
            // })
            // this.priliminaryDise.priliminaryDiseStrList = this.priliminaryDise.priliminaryDiseStrList.substr(0, this.priliminaryDise.priliminaryDiseStrList.length - 1);
            // this.priliminaryDise.priliminaryDiseIdList = this.priliminaryDise.priliminaryDiseIdList.substr(0, this.priliminaryDise.priliminaryDiseIdList.length - 1);
            // this.priliminaryDise.name = this.patient.patientName
            // this.priliminaryDise.gender = this.patient.patientSex
            // this.priliminaryDise.onsetTime = parseTime(this.priliminaryDise.onsetTime).substr(0,10)
            // this.priliminaryDise.ageStr = this.patient.patientAge
            // this.priliminaryDise.patientId=this.patient.patientId
            // submitPriliminaryDise(this.priliminaryDise).then(res=>{
            //     this.$http({
            //       url: this.$http.adornUrl(`/electronic_case/case/save`),
            //       method: 'post',
            //       data: this.$http.adornData(this.priliminaryDise)
            //       // title: '成功',
            //       // message: '成功提交初诊病历',
            //       // type: 'success',
            //       // duration: 2000
            //     }).then(({data}) => {
            //       this.confirmButtonDisabled = true
            //       if (data && data.code === 200) {
            //         this.$message({
            //           message: '操作成功',
            //           type: 'success',
            //           duration: 1000,
            //           onClose: () => {
            //             this.visible = false
            //             this.$emit('refreshDataList')
            //           }
            //         })
            //       } else {
            //         this.$message.error(data.msg)
            //       }
            //     })
            //   })
          },
          deleteZd(row) {
            this.icdZd = this.icdZd.filter(item => {
              if (item.icdId === row.icdId)
                return false
              return true
            })
          },
          selectDis(val) {
            this.$confirm('是否添加 ' + val.icdName + ' 到该患者?', '添加诊断', {
              confirmButtonText: '确认',
              cancelButtonText: '取消',
              type: 'success'
            }).then(() => {
              let flag = 1
              this.icdZd.forEach(item => {
                if (item.icdId === val.icdId) {
                  flag = 0
                }
              })
              if (flag)
                this.icdZd.push(val)
              else
                alert('已存在该诊断！')
              this.dialogTableVisible = false
            })
          },
          addIcd() {
            this.dialogTableVisible = true
            this.getIcd()
          },
          controlfast() {
            this.isclose = !this.isclose
            if (this.mainwidth === "60%")
              this.mainwidth = "80%"
            else
              this.mainwidth = "60%"
          }
        }
      }
    }
  }
</script>
<style>
  .el-scrollbar__wrap {
    overflow-x: hidden;
  }
</style>
