<template>
    <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
                <el-button v-if="isAuth('drugs_purchase:purchase:delete')" type="danger" @click="deleteHandle()"
                           :disabled="dataListSelections.length <= 0">批量删除
                </el-button>
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
                    prop="supplierId"
                    header-align="center"
                    align="center"
                    label="供应商id">
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
                    prop="purchaseState"
                    header-align="center"
                    align="center"
                    label="状态">
                <template slot-scope="scope">
                    <el-popover v-if="scope.row.purchaseState == 0" placement="top">
                        <div slot="reference">
                            <el-tag type="danger" size="medium">未审核</el-tag>
                        </div>
                    </el-popover>
                    <el-tag type="success" v-if="scope.row.purchaseState == 1">审核通过</el-tag>
                    <el-popover v-if="scope.row.purchaseState == 2" trigger="hover" placement="top">
                        <p>未通过原因: {{ scope.row.purchaseFailure }}</p>
                        <div slot="reference">
                            <el-tag type="danger" size="medium">审核未通过</el-tag>
                        </div>
                    </el-popover>
                </template>
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
                    <el-button type="text" size="small" @click="deleteHandle(scope.row.purchaseId)">删除</el-button>
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
            <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="pDetailed = false">确 定</el-button>
            </span>
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
                        'key': this.dataForm.key
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
            },
            // 删除
            deleteHandle(id) {
                var ids = id ? [id] : this.dataListSelections.map(item => {
                    return item.id
                })
                this.$confirm(`确定对这${ids.length}条数据进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                        url: this.$http.adornUrl('/drugs_purchase/purchase/delete'),
                        method: 'delete',
                        data: this.$http.adornData(ids, false)
                    }).then(({data}) => {
                        if (data && data.code === 200) {
                            this.$message({
                                message: '操作成功',
                                type: 'success',
                                duration: 1000,
                                onClose: () => {
                                    this.getDataList()
                                }
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                })
            }
        }
    }
</script>
