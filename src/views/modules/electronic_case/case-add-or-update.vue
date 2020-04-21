<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="医生id" prop="uid">
        <el-input v-model="dataForm.uid" placeholder="医生id"></el-input>
      </el-form-item>
      <el-form-item label="病人id" prop="patientId">
        <el-input v-model="dataForm.patientId" placeholder="病人id"></el-input>
      </el-form-item>
      <el-form-item label="科室id" prop="departmentId">
        <el-input v-model="dataForm.departmentId" placeholder="科室id"></el-input>
      </el-form-item>
      <el-form-item label="入院时间" prop="enterHospital">
        <el-input v-model="dataForm.enterHospital" placeholder="入院时间"></el-input>
      </el-form-item>
      <el-form-item label="出院时间" prop="leaveHospital">
        <el-input v-model="dataForm.leaveHospital" placeholder="出院时间"></el-input>
      </el-form-item>
      <el-form-item label="症状" prop="patientSymptom">
        <el-input v-model="dataForm.patientSymptom" placeholder="症状"></el-input>
      </el-form-item>
      <el-form-item label="医嘱" prop="doctorAdvice">
        <el-input v-model="dataForm.doctorAdvice" placeholder="医嘱"></el-input>
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
      this.dataForm.id = id || ''
      this.visible = true
      this.confirmButtonDisabled = false
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/electronic_case/case/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.case
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
            url: this.$http.adornUrl(`/electronic_case/case/${!this.dataForm.id ? 'save' : 'update'}`),
            method: !this.dataForm.id ? 'post' : 'put',
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
