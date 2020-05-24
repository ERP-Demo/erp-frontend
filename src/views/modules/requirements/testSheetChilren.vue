<template>
    <el-dialog
            title='项目详细'
            :close-on-click-modal="false"
            :visible.sync="visible">
        <el-table
                :data="dataForm.dataList"
                border
                v-loading="dataListLoading"
                style="width: 100%;">
            <el-table-column
                    prop="testName"
                    header-align="center"
                    align="center"
                    label="父级化验项目下的化验内容">
            </el-table-column>
            <el-table-column
                    prop="floor"
                    header-align="center"
                    align="center"
                    label="下限">
            </el-table-column>
            <el-table-column
                    prop="ceiling"
                    header-align="center"
                    align="center"
                    label="上限">
            </el-table-column>
            <el-table-column
                    prop="unit"
                    header-align="center"
                    align="center"
                    label="计量单位">
            </el-table-column>
            <el-table-column
                    header-align="center"
                    align="center"
                    label="结果">
                <template slot-scope="scope">
                    <el-input v-model="scope.row.result"></el-input>
                </template>
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

        <span slot="footer" class="dialog-footer">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        </span>

    </el-dialog>
</template>

<script>
    export default {
        data() {
            return {
                dataForm: {
                    dataList:[],
                    testAll:''
                },
                visible: false,
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataListLoading: false,
                testId:''
            }
        },
        methods: {
            init(id,tid) {
                this.testId=tid
                this.visible = true
                this.confirmButtonDisabled = false
                this.$nextTick(() => {
                    if (id) {
                        this.dataListLoading = true
                        this.$http({
                            url: this.$http.adornUrl('/requirements/testsheetChilren/selectTestSheetChilren'),
                            method: 'get',
                            params: this.$http.adornParams({
                                'page': this.pageIndex,
                                'limit': this.pageSize,
                                'pid': id
                            })
                        }).then(({data}) => {
                            if (data && data.code === 200) {
                                this.dataForm.dataList = data.page.list

                                this.totalPage = data.page.totalCount
                            } else {
                                this.dataList = []
                                this.totalPage = 0
                            }
                            this.dataListLoading = false
                        })
                    }
                })
            },
            dataFormSubmit(){
                this.dataForm.testAll=this.testId;
                this.$http({
                    url: this.$http.adornUrl('/requirements/testsheet/addTestSheet'),
                    method: 'post',
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
                        /*跳转页面*/
                        this.$router.replace({ name: 'requirements-test-sheet' })
                    } else {
                        this.$message.error(data.msg)
                    }
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
            }
        }
    }
</script>