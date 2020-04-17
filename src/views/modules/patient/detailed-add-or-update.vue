<template>
  <el-dialog
    :title="!dataForm.patientId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="患者姓名" prop="patientName">
        <el-input v-model="dataForm.patientName" placeholder="患者姓名"></el-input>
      </el-form-item>
      <el-form-item label="患者年龄" prop="patientAge">
        <el-input v-model="dataForm.patientAge" placeholder="患者年龄"></el-input>
      </el-form-item>
      <el-form-item label="患者性别" prop="patientSex">
        <el-input v-model="dataForm.patientSex" placeholder="患者性别"></el-input>
      </el-form-item>
      <el-form-item label="患者联系电话" prop="patientPhone">
        <el-input v-model="dataForm.patientPhone" placeholder="患者联系电话"></el-input>
      </el-form-item>
      <el-form-item label="患者联系地址" prop="patientAddress">
        <el-input v-model="dataForm.patientAddress" placeholder="患者联系地址"></el-input>
      </el-form-item>
      <el-form-item label="患者入院时间" prop="patientAdmission">
        <el-input v-model="dataForm.patientAdmission" placeholder="患者入院时间"></el-input>
      </el-form-item>
      <el-form-item label="诊断医生" prop="patientDoctor">
        <el-input v-model="dataForm.patientDoctor" placeholder="诊断医生"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="patientNote">
        <el-input v-model="dataForm.patientNote" placeholder="备注"></el-input>
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
        patientId: [{ required: true, message: '患者id不能为空', trigger: 'blur' }],
        patientName: [{ required: true, message: '患者姓名不能为空', trigger: 'blur' }],
        patientAge: [{ required: true, message: '患者年龄不能为空', trigger: 'blur' }],
        patientSex: [{ required: true, message: '患者性别不能为空', trigger: 'blur' }],
        patientPhone: [{ required: true, message: '患者联系电话不能为空', trigger: 'blur' }],
        patientAddress: [{ required: true, message: '患者联系地址不能为空', trigger: 'blur' }],
        patientAdmission: [{ required: true, message: '患者入院时间不能为空', trigger: 'blur' }],
        patientDoctor: [{ required: true, message: '诊断医生不能为空', trigger: 'blur' }],
        patientNote: [{ required: true, message: '备注不能为空', trigger: 'blur' }]
      }
    }
  },
  methods: {
    init (id) {
      this.dataForm.patientId = id || ''
      this.visible = true
      this.confirmButtonDisabled = false
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.patientId) {
          this.$http({
            url: this.$http.adornUrl(`/patient/patient/detailed/info/${this.dataForm.patientId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.detailed
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
            url: this.$http.adornUrl(`/patient/patient/detailed/${!this.dataForm.patientId ? 'save' : 'update'}`),
            method: !this.dataForm.patientId ? 'post' : 'put',
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
