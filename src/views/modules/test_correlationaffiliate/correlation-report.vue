<template>
    <div class="mod-config">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="getDataList()"
                 label-width="90px" style="margin-top: 40px">
            <el-form-item label="项目名称:" prop="testProjectsId">
                <el-input v-model="input" placeholder="请输入项目名称" style="width: 250px;line-height: 50px"></el-input>
                <el-button type="primary" @click="getList">添加项目值</el-button>
                <!--                <el-select v-model="dataForm.testProjectsId" placeholder="请选择" style="width: 250px;line-height: 50px">-->
                <!--                    <el-option v-for="item in testProjectsList" :key="item.testProjectsId" :label="item.testName" :value="item.testProjectsId"></el-option>-->
                <!--                </el-select>-->
            </el-form-item>
            <!--            <el-form-item label="上限" prop="floor">-->
            <!--                <el-input v-model="dataForm.floor" placeholder="上限" style="width: 250px;line-height: 50px"></el-input>-->
            <!--            </el-form-item>-->
            <!--            <el-form-item label="下限" prop="ceiling">-->
            <!--                <el-input v-model="dataForm.ceiling" placeholder="下限" style="width: 250px;line-height: 50px"></el-input>-->
            <!--            </el-form-item>-->
            <!--            <el-form-item label="计量单位" prop="unit">-->
            <!--                <el-input v-model="dataForm.unit" placeholder="单位" style="width: 250px;line-height: 50px"></el-input>-->
            <!--            </el-form-item>-->
            <!--            <el-form-item label="操作人" prop="uid">-->
            <!--                <el-input v-model="dataForm.uid" placeholder="操作人" style="width: 250px;line-height: 50px"></el-input>-->
            <!--            </el-form-item>-->

        </el-form>

        <span slot="footer" class="dialog-footer" style="margin-left: 40px">
            <el-button>取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()" :disabled="confirmButtonDisabled">确定</el-button>
        </span>
        <el-dialog
                title='子项目详情'
                :close-on-click-modal="false"
                :visible.sync="visible">
            <el-table
                    :data="dataList"
                    border
                    v-loading="dataListLoading"
                    @selection-change="selectionChangeHandle"
                    style="width: 100%;">
                <el-table-column
                        type="selection"
                        header-align="center"
                        align="center"
                        width="50">
                </el-table-column>
                <el-table-column
                        prop="testAbbreviation"
                        header-align="center"
                        align="center"
                        label="化验项目的简称">
                </el-table-column>
                <el-table-column
                        prop="testName"
                        header-align="center"
                        align="center"
                        label="化验项目的全称">
                </el-table-column>
            </el-table>
            <el-pagination
                    @size-change="sizeChangeHandle"
                    @current-change="currentChangeHandle"
                    :current-page="pageIndex"
                    :page-sizes="[10, 20, 50, 100]"
                    :page-size="pageSize"
                    :total="totalPage"
                    layout="total, sizes, prev, pager, next, jumper">
            </el-pagination>
            <span slot="footer" class="dialog-footer" style="margin-left: 40px">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()" :disabled="confirmButtonDisabled">确定</el-button>
        </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "storage-report",
        data() {
            return {
                confirmButtonDisabled: false,
                dialogProjectVisible: false,
                dataForm: {
                    testProjectsId: '',
                    floor: '',
                    ceiling: '',
                    unit: '',
                    uid: '',
                    id: ''
                },
                dataRule: {
                    testProjectsId: [{required: true, message: '编号，主键ID，主键，非空不能为空', trigger: 'blur'}],
                    floor: [{required: true, message: '上限，非空不能为空', trigger: 'blur'}],
                    ceiling: [{required: true, message: '下限，不能为空', trigger: 'blur'}],
                    unit: [{required: true, message: '单位', trigger: 'blur'}],
                    uid: [{required: true, message: '操作人不能为空', trigger: 'blur'}]
                },
                testProjectsList: [],
                dataList:[],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                visible: false
            }
        },
        activated() {
            this.getDataList()
        },
        methods: {
            // 获取数据列表
            getDataList() {
                this.$message.info("123:"+this.$route.params.id)
                this.id= this.$route.params.id
                if (this.id) {
                    console.log(this.id);
                    // this.$http({
                    //     url: this.$http.adornUrl('/test_correlationaffiliate/correlation/selectTestCorrelationId'),
                    //     method: 'get'
                    // }).then(({data}) => {
                    //     if (data && data.code === 200) {
                    //         this.testProjectsList = data.list
                    //         this.console.log("aaa" + this.testProjectsList)
                    //     } else {
                    //         this.$message.error(data.msg)
                    //     }
                    // })
                }
            },
            dataFormSubmit() {
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
                        this.dataForm.testProjectsId = "",
                            this.dataForm.floor = "",
                            this.dataForm.ceiling = "",
                            this.dataForm.unit = "",
                            this.dataForm.uid = ""
                        /*跳转页面*/
                        this.$router.replace({name: 'drugs-all-storage-report'})
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            },
            // 获取数据列表
            getList() {
                this.visible = true
                this.confirmButtonDisabled = false
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/test_projects/projects/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize
                    })
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.dataList = data.page.list
                        this.totalPage = data.page.totalCount
                    } else {
                        this.dataList = []
                        this.totalPage = 0
                    }
                    this.dataListLoading = false
                })
            },
            // 每页数
            sizeChangeHandle(val) {
                this.pageSize = val
                this.pageIndex = 1
                this.getDataList()
            },
            // 当前页
            currentChangeHandle(val) {
                this.pageIndex = val
                this.getDataList()
            },
        }
    }
</script>

<style scoped>

</style>
