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
                    fixed="right"
                    header-align="center"
                    align="center"
                    width="150"
                    label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="detHandle(scope.row.purchaseId)">查看详情</el-button>
                    <el-button type="text" size="small" @click="tuiHuo(scope.row.purchaseId)">退货</el-button>
                </template>
            </el-table-column>
            <purchase ref="purchase"></purchase>
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
        <pDetailed v-if="pDetailedVisible" ref="pDetailed" @refreshDataList="getDataList"></pDetailed>
        <tuihuo v-if="tuihuoVisible" ref="tuihuo" @refreshDataList="getDataList"></tuihuo>
    </div>
</template>

<script>
    import purchase from '../drugs_purchase/purchase'
    import pDetailed from "../drugs_purchase/pDetailed";
    import tuihuo from "../drugs_purchase/tuihuo";

    export default {
        data() {
            return {
                dataForm: {
                    key: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                pDetailedVisible: false,
                tuihuoVisible: false,
                dataListLoading: false,
                dataListSelections: [],
                addOrUpdateVisible: false
            }
        },
        components: {
            purchase,
            pDetailed,
            tuihuo
        },
        activated() {
            this.getDataList()
        },
        methods: {
            // 获取数据列表
            getDataList() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/warehouseCheck'),
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
            //查看详细
            detHandle(id) {
                this.pDetailedVisible = true
                this.$nextTick(() => {
                    this.$refs.pDetailed.init(id)
                })
            },
            //退货
            tuiHuo(id) {
                this.tuihuoVisible = true
                this.$nextTick(() => {
                    this.$refs.tuihuo.init(id)
                })
                /*console.log(id)
                this.$router.push({path:'/warehouse/tuihuo/'+id})*/
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
                    this.$refs.pDetailed.init(id)
                })
            },
            //获取药
            getData() {
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
            rejHandle(id) {
                var ids = id ? [id] : this.dataListSelections.map(item => {
                    return item.processInstanceId
                })
                this.$confirm(`确定对这${ids.length}条数据进行[${id ? '驳回' : '批量驳回'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$prompt('输入原因', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        inputPattern: /^[\s\S]*.*[^\s][\s\S]*$/,
                        inputErrorMessage: '请输入原因'
                    }).then(({value}) => {
                        this.$http({
                            url: this.$http.adornUrl('/drugs_purchase/purchase/reject'),
                            method: 'post',
                            data: this.$http.adornData({
                                'ids': ids,
                                'reason': value
                            }, false)
                        }).then(({data}) => {
                            if (data && data.code === 200) {
                                this.$message.success("操作成功")
                                this.getDataList()
                            } else {
                                this.$message.error(data.msg)
                            }
                        })
                    })
                })
            },
            agreHandle(id) {
                var ids = id ? [id] : this.dataListSelections.map(item => {
                    return item.processInstanceId
                })
                this.$confirm(`确定对这${ids.length}条数据进行[${id ? '同意' : '批量同意'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                        url: this.$http.adornUrl('/drugs_purchase/purchase/agree'),
                        method: 'post',
                        data: this.$http.adornData(ids, false)
                    }).then(({data}) => {
                        if (data && data.code === 200) {
                            this.$message.success("操作成功")
                            this.getDataList()
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                })
            }
        }
    }
</script>
