<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="" prop="parentId">
        <el-input v-model="dataForm.parentId" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="level">
        <el-input v-model="dataForm.level" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="type">
        <el-input v-model="dataForm.type" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="status">
        <el-input v-model="dataForm.status" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="modelId">
        <el-input v-model="dataForm.modelId" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="scope">
        <el-input v-model="dataForm.scope" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="ownId">
        <el-input v-model="dataForm.ownId" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="createTime">
        <el-input v-model="dataForm.createTime" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="" prop="name">
        <el-input v-model="dataForm.name" placeholder=""></el-input>
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
        parentId: [{ required: true, message: '不能为空', trigger: 'blur' }],
        level: [{ required: true, message: '不能为空', trigger: 'blur' }],
        type: [{ required: true, message: '不能为空', trigger: 'blur' }],
        status: [{ required: true, message: '不能为空', trigger: 'blur' }],
        modelId: [{ required: true, message: '不能为空', trigger: 'blur' }],
        scope: [{ required: true, message: '不能为空', trigger: 'blur' }],
        ownId: [{ required: true, message: '不能为空', trigger: 'blur' }],
        createTime: [{ required: true, message: '不能为空', trigger: 'blur' }],
        name: [{ required: true, message: '不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`electronic_case_model_catalog/case/info/${this.dataForm.id}`),
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
            url: this.$http.adornUrl(`electronic_case_model_catalog/case/${!this.dataForm.id ? 'save' : 'update'}`),
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
