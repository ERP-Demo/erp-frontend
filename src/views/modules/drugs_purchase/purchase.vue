<template>
    <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
            </el-form-item>
        </el-form>
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
                    prop="purchaseId"
                    header-align="center"
                    align="center"
                    label="单号">
            </el-table-column>
            <el-table-column
                    prop="supplierName"
                    header-align="center"
                    align="center"
                    label="供应商">
            </el-table-column>
            <el-table-column
                    prop="purchaseAmountPayable"
                    header-align="center"
                    align="center"
                    label="购入应付金额">
            </el-table-column>
            <el-table-column
                    prop="purchaseActualAmountPaid"
                    header-align="center"
                    align="center"
                    label="购入实付金额">
            </el-table-column>

            <el-table-column
                    prop="purchaseTime"
                    header-align="center"
                    align="center"
                    label="购入日期">
            </el-table-column>
            <el-table-column
                    fixed="right"
                    header-align="center"
                    align="center"
                    width="150"
                    label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="pDetailed = true,addHandle(scope.row.purchaseId)">查看详情</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-dialog
                title="查看详情"
                :visible.sync="pDetailed"
                width="66%">
            <el-table
                    ref="multipleTable"
                    :data="tableData"
                    tooltip-effect="dark"
                    style="width: 100%"
                    @selection-change="handleSelectionChange">
                <el-table-column
                        prop="drugsName"
                        header-align="center"
                        align="center"
                        label="药品名称">
                </el-table-column>
                <el-table-column
                        prop="pdMoney"
                        header-align="center"
                        align="center"
                        label="单价">
                </el-table-column>
                <el-table-column
                        prop="pdNum"
                        header-align="center"
                        align="center"
                        label="数量">
                </el-table-column>
            </el-table>
        </el-dialog>
        <el-pagination
                @size-change="sizeChangeHandle"
                @current-change="currentChangeHandle"
                :current-page="pageIndex"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="pageSize"
                :total="totalPage"
                layout="total, sizes, prev, pager, next, jumper">
        </el-pagination>
        <!-- 弹窗, 新增 / 修改 -->
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    </div>
</template>

<script>
    import AddOrUpdate from './purchase-add-or-update'

    export default {
        data() {
            return {
                dataForm: {
                    key: ''
                },
                dataList: [],
                tableData: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                pDetailed: false,
                dataListLoading: false,
                dataListSelections: [],
                addOrUpdateVisible: false
            }
        },
        components: {
            AddOrUpdate
        },
        activated() {
            this.getDataList()
        },
        methods: {
            // 获取数据列表
            getDataList() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'purchaseId': this.dataForm.key
                    })
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.dataList = data.page.list
                        console.log(this.dataList)
                        this.totalPage = data.page.totalCount
                    } else {
                        this.dataList = []
                        this.totalPage = 0
                    }
                    this.dataListLoading = false
                })
            },
            addHandle(pid){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/all/'+pid),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.tableData = data.all
                    } else {
                        this.tableData = []
                    }
                    this.dataListLoading = false
                })
            },
            //获取复选框
            handleSelectionChange(val) {
                this.multipleSelection = val;
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
            // 多选
            selectionChangeHandle(val) {
                this.dataListSelections = val
            },
            // 新增 / 修改
            addOrUpdateHandle(id) {
                this.addOrUpdateVisible = true
                this.$nextTick(() => {
                    this.$refs.addOrUpdate.init(id)
                })
            },
            //获取药
            getData () {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/drugs/detailed/list'),
                    method: 'get',
                    params: this.$http.adornParams({})
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.tableData = data.page.list
                    } else {
                        this.dataList = []
                    }
                    this.dataListLoading = false
                })
            }
        }
    }
</script>
