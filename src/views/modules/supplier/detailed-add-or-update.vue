<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="供应商名称，非空" prop="supplierName">
        <el-input v-model="dataForm.supplierName" placeholder="供应商名称，非空"></el-input>
      </el-form-item>
      <el-form-item label="联系电话" prop="supplierCartPhone">
        <el-input v-model="dataForm.supplierCartPhone" placeholder="联系电话"></el-input>
      </el-form-item>
      <el-form-item label="供应商类型：1.自营，2.平台" prop="supplierType">
        <el-input v-model="dataForm.supplierType" placeholder="供应商类型：1.自营，2.平台"></el-input>
      </el-form-item>
      <el-form-item label="供应商联系人" prop="supplierMan">
        <el-input v-model="dataForm.supplierMan" placeholder="供应商联系人"></el-input>
      </el-form-item>
      <el-form-item label="供应商开户银行名称" prop="supplierBankName">
        <el-input v-model="dataForm.supplierBankName" placeholder="供应商开户银行名称"></el-input>
      </el-form-item>
      <el-form-item label="银行账号" prop="supplierBankAccount">
        <el-input v-model="dataForm.supplierBankAccount" placeholder="银行账号"></el-input>
      </el-form-item>
      <el-form-item label="供应商地址" prop="supplierAddress">
        <el-input v-model="dataForm.supplierAddress" placeholder="供应商地址"></el-input>
      </el-form-item>
      <el-form-item label="状态：0禁止，1启用" prop="supplierStatus">
        <el-input v-model="dataForm.supplierStatus" placeholder="状态：0禁止，1启用"></el-input>
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
        supplierId: [{ required: true, message: '供应商的编号，主键ID，主键，非空不能为空', trigger: 'blur' }],
        supplierName: [{ required: true, message: '供应商名称，非空不能为空', trigger: 'blur' }],
        supplierCartPhone: [{ required: true, message: '联系电话不能为空', trigger: 'blur' }],
        supplierType: [{ required: true, message: '供应商类型：1.自营，2.平台不能为空', trigger: 'blur' }],
        supplierMan: [{ required: true, message: '供应商联系人不能为空', trigger: 'blur' }],
        supplierBankName: [{ required: true, message: '供应商开户银行名称不能为空', trigger: 'blur' }],
        supplierBankAccount: [{ required: true, message: '银行账号不能为空', trigger: 'blur' }],
        supplierAddress: [{ required: true, message: '供应商地址不能为空', trigger: 'blur' }],
        supplierStatus: [{ required: true, message: '状态：0禁止，1启用不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`/supplier/detailed/info/${this.dataForm.id}`),
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
            url: this.$http.adornUrl(`/supplier/detailed/${!this.dataForm.id ? 'save' : 'update'}`),
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
