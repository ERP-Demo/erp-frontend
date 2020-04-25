<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="供应商的编号" prop="supplierId">
        <el-input v-model="dataForm.supplierId" placeholder="供应商的编号"></el-input>
      </el-form-item>
      <el-form-item label="购入应付金额" prop="purchaseAmountPayable">
        <el-input v-model="dataForm.purchaseAmountPayable" placeholder="购入应付金额"></el-input>
      </el-form-item>
      <el-form-item label="购入实付金额" prop="purchaseActualAmountPaid">
        <el-input v-model="dataForm.purchaseActualAmountPaid" placeholder="购入实付金额"></el-input>
      </el-form-item>
      <el-form-item label="审核 0是未审核 1是审核通过 2是审核未通过" prop="purchaseState">
        <el-input v-model="dataForm.purchaseState" placeholder="审核 0是未审核 1是审核通过 2是审核未通过"></el-input>
      </el-form-item>
      <el-form-item label="审核未通过原因" prop="purchaseFailure">
        <el-input v-model="dataForm.purchaseFailure" placeholder="审核未通过原因"></el-input>
      </el-form-item>
      <el-form-item label="购入日期" prop="purchaseTime">
        <el-input v-model="dataForm.purchaseTime" placeholder="购入日期"></el-input>
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
        purchaseId: [{ required: true, message: '药品购入编号，主键ID，主键，非空不能为空', trigger: 'blur' }],
        supplierId: [{ required: true, message: '供应商的编号不能为空', trigger: 'blur' }],
        purchaseAmountPayable: [{ required: true, message: '购入应付金额不能为空', trigger: 'blur' }],
        purchaseActualAmountPaid: [{ required: true, message: '购入实付金额不能为空', trigger: 'blur' }],
        purchaseState: [{ required: true, message: '审核 0是未审核 1是审核通过 2是审核未通过不能为空', trigger: 'blur' }],
        purchaseFailure: [{ required: true, message: '审核未通过原因不能为空', trigger: 'blur' }],
        purchaseTime: [{ required: true, message: '购入日期不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`/drugs_purchase/purchase/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.purchase
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
            url: this.$http.adornUrl(`drugs_purchase/purchase/${!this.dataForm.id ? 'save' : 'update'}`),
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
