<template>
    <div class="mod-config">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="getDataList()" label-width="90px" style="margin-top: 40px">
            <el-form-item label="药品:" prop="drugsId">
                <el-select v-model="dataForm.drugsId" placeholder="请选择药品" style="width: 250px;line-height: 50px">
                    <el-option v-for="item in drugsList" :key="item.drugsId" :label="item.drugsName" :value="item.drugsId"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="数量" prop="reportedLossCount">
                <el-input v-model="dataForm.reportedLossCount" placeholder="数量" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
            <el-form-item label="报损原因" prop="reportedLossWhy">
                <el-input v-model="dataForm.reportedLossWhy" placeholder="报损原因" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
            <el-form-item label="备注" prop="reportedLossNote">
                <el-input v-model="dataForm.reportedLossNote" placeholder="备注" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
            <el-form-item label="操作人" prop="reportedLossOperationOf">
                <el-input v-model="dataForm.reportedLossOperationOf" placeholder="操作人" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer" style="margin-left: 40px">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()" :disabled="confirmButtonDisabled">确定</el-button>
        </span>
    </div>
</template>

<script>
    import AddOrUpdate from "@/views/modules/drugs/detailed-add-or-update";

    export default {
        name: "storage-report",
        data () {
            return {
                confirmButtonDisabled: false,
                dataForm: {
                    drugsId: '',
                    reportedLossCount: '',
                    reportedLossWhy: '',
                    reportedLossNote: '',
                    reportedLossOperationOf: ''
                },
                dataRule: {
                    drugsId: [{ required: true, message: '药品编号，主键ID，主键，非空不能为空', trigger: 'blur' }],
                    reportedLossCount: [{ required: true, message: '数量，非空不能为空', trigger: 'blur' }],
                    reportedLossWhy: [{ required: true, message: '报损原因不能为空', trigger: 'blur' }],
                    reportedLossNote: [{ required: true, message: '备注不能为空', trigger: 'blur' }],
                    reportedLossOperationOf: [{ required: true, message: '操作人不能为空', trigger: 'blur' }]
                },
                drugsList: []
            }
        },
        activated () {
            this.getDataList()
        },
        methods: {
            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/drugs/reports/selectDrugsId'),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.drugsList = data.list
                        console.log("aaa"+this.drugsList)
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            },
            dataFormSubmit(){
                this.$http({
                    url: this.$http.adornUrl('/drugs/reports/addStorageReport'),
                    method: 'post',
                    data: this.$http.adornData(this.dataForm)
                }).then(({data}) => {
                    this.confirmButtonDisabled = true
                    if (data && data.code === 200) {
                        this.$message({
                            message: '操作成功',
                            type: 'success',
                            duration: 1000
                        })
                        /*刷新页面*/
                        this.dataForm.drugsId="",
                        this.dataForm.reportedLossCount="",
                        this.dataForm.reportedLossWhy="",
                        this.dataForm.reportedLossNote="",
                        this.dataForm.reportedLossOperationOf=""
                        /*跳转页面*/
                        this.$router.replace({ name: 'drugs-all-storage-report' })
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            }
        }
    }
</script>

<style scoped>

</style>