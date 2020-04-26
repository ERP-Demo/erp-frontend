<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item label="综合（父级）化验项目名" prop="testSynthesizeId">
        <el-input v-model="dataForm.testSynthesizeId" placeholder="综合（父级）化验项目名"></el-input>
      </el-form-item>
      <el-form-item label="父级化验项目下的化验内容" prop="testProjectsId">
        <el-input v-model="dataForm.testProjectsId" placeholder="父级化验项目下的化验内容"></el-input>
      </el-form-item>
      <el-form-item label="下限" prop="floor">
        <el-input v-model="dataForm.floor" placeholder="下限"></el-input>
      </el-form-item>
      <el-form-item label="上限" prop="ceiling">
        <el-input v-model="dataForm.ceiling" placeholder="上限"></el-input>
      </el-form-item>
      <el-form-item label="计量单位" prop="unit">
        <el-input v-model="dataForm.unit" placeholder="计量单位"></el-input>
      </el-form-item>
      <el-form-item label="创建时间" prop="createtime">
        <el-input v-model="dataForm.createtime" placeholder="创建时间"></el-input>
      </el-form-item>
      <el-form-item label="创建者" prop="uid">
        <el-input v-model="dataForm.uid" placeholder="创建者"></el-input>
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
        testSynthesizeId: [{ required: true, message: '综合（父级）化验项目名不能为空', trigger: 'blur' }],
        testProjectsId: [{ required: true, message: '父级化验项目下的化验内容不能为空', trigger: 'blur' }],
        floor: [{ required: true, message: '下限不能为空', trigger: 'blur' }],
        ceiling: [{ required: true, message: '上限不能为空', trigger: 'blur' }],
        unit: [{ required: true, message: '计量单位不能为空', trigger: 'blur' }],
        createtime: [{ required: true, message: '创建时间不能为空', trigger: 'blur' }],
        uid: [{ required: true, message: '创建者不能为空', trigger: 'blur' }]
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
            url: this.$http.adornUrl(`/test_correlationaffiliate/correlation/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.dataForm = data.correlation
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
            url: this.$http.adornUrl(`/test_correlationaffiliate/correlation/${!this.dataForm.id ? 'save' : 'update'}`),
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
