<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="化验模板名称" prop="testModelName">
        <el-input v-model="dataForm.testModelName" placeholder="化验模板名称"></el-input>
      </el-form-item>
      <el-form-item label="化验模板简介" prop="testModelIntroduction">
        <el-input v-model="dataForm.testModelIntroduction" placeholder="化验模板简介"></el-input>
      </el-form-item>
      <el-form-item label="化验模板的编码" prop="testModelCode">
        <el-input v-model="dataForm.testModelCode" placeholder="化验模板的编码"></el-input>
      </el-form-item>
      <el-form-item label="化验模板的范围(0:个人,1:科室,2:全院)" prop="testModelRange">
        <el-input v-model="dataForm.testModelRange" placeholder="化验模板的范围(0:个人,1:科室,2:全院)"></el-input>
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
        testModelId: [{ required: true, message: '化验模板的编号不能为空', trigger: 'blur' }],
        testModelName: [{ required: true, message: '化验模板名称不能为空', trigger: 'blur' }],
        testModelIntroduction: [{ required: true, message: '化验模板简介不能为空', trigger: 'blur' }],
        testModelCode: [{ required: true, message: '化验模板的编码不能为空', trigger: 'blur' }],
        testModelRange: [{ required: true, message: '化验模板的范围(0:个人,1:科室,2:全院)不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`test_model/model/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.model
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
            url: this.$http.adornUrl(`test_model/model/${!this.dataForm.id ? 'save' : 'update'}`),
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
