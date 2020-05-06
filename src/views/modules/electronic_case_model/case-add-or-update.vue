<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="" prop="chiefComplaint">
        <el-input v-model="dataForm.chiefComplaint" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="historyOfPresentIllness">
        <el-input v-model="dataForm.historyOfPresentIllness" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="historyOfTreatment">
        <el-input v-model="dataForm.historyOfTreatment" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="pastHistory">
        <el-input v-model="dataForm.pastHistory" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="allergies">
        <el-input v-model="dataForm.allergies" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="healthCheckup">
        <el-input v-model="dataForm.healthCheckup" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="priliminaryDiseIdList">
        <el-input v-model="dataForm.priliminaryDiseIdList" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="priliminaryDiseStrList">
        <el-input v-model="dataForm.priliminaryDiseStrList" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="name">
        <el-input v-model="dataForm.name" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="status">
        <el-input v-model="dataForm.status" placeholder=""></el-input>
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
        id: [{ required: true, message: '不能为空', trigger: 'blur' }],
        chiefComplaint: [{ required: true, message: '不能为空', trigger: 'blur' }],
        historyOfPresentIllness: [{ required: true, message: '不能为空', trigger: 'blur' }],
        historyOfTreatment: [{ required: true, message: '不能为空', trigger: 'blur' }],
        pastHistory: [{ required: true, message: '不能为空', trigger: 'blur' }],
        allergies: [{ required: true, message: '不能为空', trigger: 'blur' }],
        healthCheckup: [{ required: true, message: '不能为空', trigger: 'blur' }],
        priliminaryDiseIdList: [{ required: true, message: '不能为空', trigger: 'blur' }],
        priliminaryDiseStrList: [{ required: true, message: '不能为空', trigger: 'blur' }],
        name: [{ required: true, message: '不能为空', trigger: 'blur' }],
        status: [{ required: true, message: '不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`electronic_case_model/case/info/${this.dataForm.id}`),
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
            url: this.$http.adornUrl(`electronic_case_model/case/${!this.dataForm.id ? 'save' : 'update'}`),
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
