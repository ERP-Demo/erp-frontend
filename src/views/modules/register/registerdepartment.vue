<template>
  <div class="mod-config">
    <h2>挂号登记</h2>
    <el-form :model="dataForm" :inline="true" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="85px" style="margin-top: 40px;">
      <el-form-item label="患者名称" prop="patientName">
        <el-input placeholder="患者名称" v-model="dataForm.patientName" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
      <el-form-item label="患者性别" prop="patientSex">
        <el-radio-group v-model="dataForm.patientSex" style="width: 250px;line-height: 50px;">
          <el-radio label="男"></el-radio>
          <el-radio label="女"></el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="患者年龄" prop="patientAge">
        <el-input placeholder="患者年龄" v-model="dataForm.patientAge" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
      <el-form-item label="身份证" prop="patientCartnum">
        <el-input placeholder="身份证" v-model="dataForm.patientCartnum" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
      <el-form-item label="患者电话" prop="patientPhone">
        <el-input placeholder="患者电话" v-model="dataForm.patientPhone" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
      <el-form-item label="患者地址" prop="patientAddress">
        <el-input placeholder="患者地址" v-model="dataForm.patientAddress" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
      <el-form-item label="科室" prop="departmentId">
        <el-select placeholder="请选择" v-model="dataForm.departmentId" style="width: 250px;line-height: 50px;">
          <el-option
                  v-for="item in dlist"
                  :key="item.departmentId"
                  :label="item.departmentName"
                  :value="item.departmentId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="患者备注" prop="patientNote">
        <el-input placeholder="患者备注" v-model="dataForm.patientNote" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
      <el-form-item label="挂号费用" prop="registerCost">
        <el-input placeholder="挂号费用" v-model="dataForm.registerCost" style="width: 250px;line-height: 50px;"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer" style="margin-left: 450px;">
      <el-button type="primary" @click="dataFormSubmit()" :disabled="confirmButtonDisabled">确定</el-button>
    </span>

  </div>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        confirmButtonDisabled: false,
        dataForm: {},
        dlist:{},
        dataRule: {
          patientName: [{ required: true, message: '不能为空', trigger: 'blur' }],
          patientAge: [{ required: true, message: '不能为空', trigger: 'blur' }],
          patientSex: [{ required: true, message: '不能为空', trigger: 'blur' }],
          patientPhone: [{ required: true, message: '不能为空', trigger: 'blur' }],
          patientAddress: [{ required: true, message: '不能为空', trigger: 'blur' }],
          patientNote: '',
          registerCost: [{ required: true, message: '不能为空', trigger: 'blur' }],
          departmentId: [{ required: true, message: '不能为空', trigger: 'blur' }],
          patientCartnum:[{ required: true, message: '不能为空', trigger: 'blur' }]
        }
      }
    },
    activated () {
      this.init()
    },
    methods: {
      init () {
        this.$http({
          url: this.$http.adornUrl('/department/Department/getAll'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.dlist = data.list
            console.log(this.dlist)
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        // this.$refs['dataForm'].validate((valid) => {
        //     if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/register/register/${!this.dataForm.registerId ? 'save' : 'update'}`),
              method: !this.dataForm.registerId ? 'post' : 'put',
              data: this.$http.adornData(this.dataForm)
            }).then(({data}) => {
              this.confirmButtonDisabled = true
              if (data && data.code === 200) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1000,
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
       // })
      //}
    }
  }
</script>