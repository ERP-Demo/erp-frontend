<template>
    <div class="mod-config">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="getDataList()" label-width="90px" style="margin-top: 40px">
            <el-form-item label="子化验项目项目:" prop="testProjectsId">
                <el-select v-model="dataForm.testProjectsId" placeholder="请选择" style="width: 250px;line-height: 50px">
                    <el-option v-for="item in testProjectsList" :key="item.testProjectsId" :label="item.testName" :value="item.testProjectsId"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="上限" prop="floor">
                <el-input v-model="dataForm.floor" placeholder="上限" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
            <el-form-item label="下限" prop="ceiling">
                <el-input v-model="dataForm.ceiling" placeholder="下限" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
            <el-form-item label="计量单位" prop="unit">
                <el-input v-model="dataForm.unit" placeholder="单位" style="width: 250px;line-height: 50px"></el-input>
            </el-form-item>
            <el-form-item label="操作人" prop="uid">
                <el-input v-model="dataForm.uid" placeholder="操作人" style="width: 250px;line-height: 50px"></el-input>
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
                    testProjectsId: '',
                    floor: '',
                    ceiling: '',
                    unit: '',
                    uid: ''
                },
                dataRule: {
                    testProjectsId: [{ required: true, message: '编号，主键ID，主键，非空不能为空', trigger: 'blur' }],
                    floor: [{ required: true, message: '上限，非空不能为空', trigger: 'blur' }],
                    ceiling: [{ required: true, message: '下限，不能为空', trigger: 'blur' }],
                    unit: [{ required: true, message: '单位', trigger: 'blur' }],
                    uid: [{ required: true, message: '操作人不能为空', trigger: 'blur' }]
                },
                testProjectsList: []
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
                    url: this.$http.adornUrl('/test_correlationaffiliate/correlation/selectTestCorrelationId'),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.testProjectsList = data.list
                        console.log("aaa"+this.testProjectsList)
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            },
            dataFormSubmit(){
                this.$http({
                    url: this.$http.adornUrl('/test_correlationaffiliate/correlation/addTestCorrelation'),
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
                        this.dataForm.testProjectsId="",
                            this.dataForm.floor="",
                            this.dataForm.ceiling="",
                            this.dataForm.unit="",
                            this.dataForm.uid=""
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