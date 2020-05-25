<template>
  <el-dialog
  :title="!dataForm.caseNo ? '新增' : '修改'"
  :close-on-click-modal="false"
  :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="医生id" prop="uid">
        <el-input v-model="dataForm.uid" placeholder="医生id"></el-input>
      </el-form-item>
      <el-form-item label="病人id" prop="patientId">
        <el-input v-model="dataForm.patientId" placeholder="病人id"></el-input>
      </el-form-item>
      <el-form-item label="主诉" prop="complain">
        <el-input v-model="dataForm.complain" placeholder="主诉"></el-input>
      </el-form-item>
      <el-form-item label="发病时间" prop="onsetTime">
        <el-input v-model="dataForm.onsetTime" placeholder="发病时间"></el-input>
      </el-form-item>
      <el-form-item label="症状" prop="patientSymptom">
        <el-input v-model="dataForm.patientSymptom" placeholder="症状"></el-input>
      </el-form-item>
      <el-form-item label="既往病史" prop="patientSymptom">
        <el-input v-model="dataForm.patientSymptom" placeholder="既往病史"></el-input>
      </el-form-item>
      <el-form-item label="过敏史" prop="allergyHistory">
        <el-input v-model="dataForm.allergyHistory" placeholder="过敏史"></el-input>
      </el-form-item>
      <el-form-item label="体格检查" prop="healthCheckup">
        <el-input v-model="dataForm.healthCheckup" placeholder="体格检查"></el-input>
      </el-form-item>
      <el-form-item label="治疗情况" prop="treatment">
        <el-input v-model="dataForm.treatment" placeholder="治疗情况"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()" :disabled="confirmButtonDisabled">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data () {
    return {
      visible: false,
      confirmButtonDisabled: false,
      dataForm: {},
      dataRule: {
        caseNo: [{ required: true, message: '病例id不能为空', trigger: 'blur' }],
        uid: [{ required: true, message: '医生id不能为空', trigger: 'blur' }],
        patientId: [{ required: true, message: '病人id不能为空', trigger: 'blur' }],
        departmentId: [{ required: true, message: '科室id不能为空', trigger: 'blur' }],
        enterHospital: [{ required: true, message: '入院时间不能为空', trigger: 'blur' }],
        leaveHospital: [{ required: true, message: '出院时间不能为空', trigger: 'blur' }],
        patientSymptom: [{ required: true, message: '症状不能为空', trigger: 'blur' }],
        doctorAdvice: [{ required: true, message: '医嘱不能为空', trigger: 'blur' }]
      }
    }
  },
  methods: {
    init (id) {
      this.dataForm.caseNo = id || ''
      console.log(this.dataForm.caseNo)
      this.visible = true
      this.confirmButtonDisabled = false
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.caseNo) {
          this.$http({
            url: this.$http.adornUrl(`/electronic_case/case/info/${this.dataForm.caseNo}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.e
            }
          })
        } else {
          this.dataForm = {}
        }
      })
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/electronic_case/case/${!this.dataForm.caseNo ? 'save' : 'update'}`),
            method: !this.dataForm.caseNo ? 'post' : 'put',
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
        }
      })
    }
  }
}
</script>
