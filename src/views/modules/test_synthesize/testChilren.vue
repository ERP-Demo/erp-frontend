<template>
    <el-dialog
            title='项目详细'
            :close-on-click-modal="false"
            :visible.sync="visible">
        <el-table
                :data="dataList"
                border
                v-loading="dataListLoading"
                @selection-change="selectionChangeHandle"
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
    </el-dialog>
</template>

<script>
    export default {
        data() {
            return {
                visible: false,
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataList: [],
                dataListLoading: false
            }
        },
        methods: {
            init(id) {
                this.visible = true
                this.confirmButtonDisabled = false
                this.$nextTick(() => {
                    if (id) {
                        this.dataListLoading = true
                        this.$http({
                            url: this.$http.adornUrl('/test_correlation/correlation/list'),
                            method: 'get',
                            params: this.$http.adornParams({
                                'page': this.pageIndex,
                                'limit': this.pageSize,
                                'pid': id
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
            },
            updateHandle(id) {
                this.$router.push({
                    name: 'test_correlationaffiliate-correlation-report',
                    params: {
                        id: id
                    }
                })
            }
        }
    }
</script>