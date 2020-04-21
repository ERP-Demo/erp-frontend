<template>
  <el-dialog
    :title="!dataForm.registerId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="患者名称" prop="patientName">
        <el-input v-model="dataForm.patientName" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="是否退号" prop="isBack">
        <el-input v-model="dataForm.isBack" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="挂号费用" prop="registerCost">
        <el-input v-model="dataForm.registerCost" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="科室编号" prop="departmentId">
        <el-input v-model="dataForm.departmentId" placeholder=""></el-input>
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
        registerId: [{ required: true, message: '不能为空', trigger: 'blur' }],
        patientName: [{ required: true, message: '不能为空', trigger: 'blur' }],
        isBack: [{ required: true, message: '不能为空', trigger: 'blur' }],
        registerCost: [{ required: true, message: '不能为空', trigger: 'blur' }],
        departmentId: [{ required: true, message: '不能为空', trigger: 'blur' }]
      }
    }
  },
  methods: {
    init (id) {
      this.dataForm.registerId = id || ''
      this.visible = true
      this.confirmButtonDisabled = false
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.registerId) {
          this.$http({
            url: this.$http.adornUrl(`/register/register/info/${this.dataForm.registerId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.register
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
            url: this.$http.adornUrl(`register/register/${!this.dataForm.registerId ? 'save' : 'update'}`),
            method: !this.dataForm.registerId ? 'post' : 'put',
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
