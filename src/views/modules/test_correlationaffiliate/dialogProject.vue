<template>
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
</template>

<script>
    export default {
        data() {
            return {
                dataLis:[],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                visible: false
            }
        },
        activated() {
            this.getList()
        },
        methods: {
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
