<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="项目编码" prop="projectId">
        <el-input v-model="dataForm.projectId" placeholder="项目编码"></el-input>
      </el-form-item>
      <el-form-item label="项目名称" prop="projectName">
        <el-input v-model="dataForm.projectName" placeholder="项目名称"></el-input>
      </el-form-item>
      <el-form-item label="目的" prop="purpose">
        <el-input v-model="dataForm.purpose" placeholder="目的"></el-input>
      </el-form-item>
      <el-form-item label="要求" prop="requirements">
        <el-input v-model="dataForm.requirements" placeholder="要求"></el-input>
      </el-form-item>
      <el-form-item label="临床印象" prop="clinicalImpression">
        <el-input v-model="dataForm.clinicalImpression" placeholder="临床印象"></el-input>
      </el-form-item>
      <el-form-item label="临床诊断" prop="clinicalDiagnosis">
        <el-input v-model="dataForm.clinicalDiagnosis" placeholder="临床诊断"></el-input>
      </el-form-item>
      <el-form-item label="检查部位" prop="checkThe">
        <el-input v-model="dataForm.checkThe" placeholder="检查部位"></el-input>
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
        projectId: [{ required: true, message: '项目编码不能为空', trigger: 'blur' }],
        projectName: [{ required: true, message: '项目名称不能为空', trigger: 'blur' }],
        purpose: [{ required: true, message: '目的不能为空', trigger: 'blur' }],
        requirements: [{ required: true, message: '要求不能为空', trigger: 'blur' }],
        clinicalImpression: [{ required: true, message: '临床印象不能为空', trigger: 'blur' }],
        clinicalDiagnosis: [{ required: true, message: '临床诊断不能为空', trigger: 'blur' }],
        checkThe: [{ required: true, message: '检查部位不能为空', trigger: 'blur' }],
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
            url: this.$http.adornUrl(`/requirements/requirements/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.requirements
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
            url: this.$http.adornUrl(`/requirements/requirements/${!this.dataForm.id ? 'save' : 'update'}`),
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
