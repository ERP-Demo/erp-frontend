<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="药品名称，非空" prop="drugsName">
        <el-input v-model="dataForm.drugsName" placeholder="药品名称，非空"></el-input>
      </el-form-item>
      <el-form-item label="药品单价" prop="drugsPrice">
        <el-input v-model="dataForm.drugsPrice" placeholder="药品单价"></el-input>
      </el-form-item>
      <el-form-item label="药品规格" prop="drugsNorms">
        <el-input v-model="dataForm.drugsNorms" placeholder="药品规格"></el-input>
      </el-form-item>
      <el-form-item label="药品用法" prop="drugsUsage">
        <el-input v-model="dataForm.drugsUsage" placeholder="药品用法"></el-input>
      </el-form-item>
      <el-form-item label="药品用量" prop="drugsDosage">
        <el-input v-model="dataForm.drugsDosage" placeholder="药品用量"></el-input>
      </el-form-item>
      <el-form-item label="禁忌" prop="drugsTaboo">
        <el-input v-model="dataForm.drugsTaboo" placeholder="禁忌"></el-input>
      </el-form-item>
      <el-form-item label="生产厂商" prop="drugsProducer">
        <el-input v-model="dataForm.drugsProducer" placeholder="生产厂商"></el-input>
      </el-form-item>
      <el-form-item label="批准文号" prop="drugsApprovalNumber">
        <el-input v-model="dataForm.drugsApprovalNumber" placeholder="批准文号"></el-input>
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
        drugsId: [{ required: true, message: '药品编号，主键ID，主键，非空不能为空', trigger: 'blur' }],
        drugsName: [{ required: true, message: '药品名称，非空不能为空', trigger: 'blur' }],
        drugsPrice: [{ required: true, message: '药品单价不能为空', trigger: 'blur' }],
        drugsNorms: [{ required: true, message: '药品规格不能为空', trigger: 'blur' }],
        drugsUsage: [{ required: true, message: '药品用法不能为空', trigger: 'blur' }],
        drugsDosage: [{ required: true, message: '药品用量不能为空', trigger: 'blur' }],
        drugsTaboo: [{ required: true, message: '禁忌不能为空', trigger: 'blur' }],
        drugsProducer: [{ required: true, message: '生产厂商不能为空', trigger: 'blur' }],
        drugsApprovalNumber: [{ required: true, message: '批准文号不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`/drugs/detailed/info/${this.dataForm.id}`),
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
            url: this.$http.adornUrl(`/drugs/detailed/${!this.dataForm.id ? 'save' : 'update'}`),
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
