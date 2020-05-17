<template>
    <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.key" placeholder="进货单号" clearable></el-input>
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
                    prop="tuihuoId"
                    header-align="center"
                    align="center"
                    label="退货单号">
            </el-table-column>
            <el-table-column
                    prop="purchaseId"
                    header-align="center"
                    align="center"
                    label="进货单号">
            </el-table-column>
            <el-table-column
                    prop="supplierName"
                    header-align="center"
                    align="center"
                    label="供应商">
            </el-table-column>
            <el-table-column
                    prop="username"
                    header-align="center"
                    align="center"
                    label="操作人">
            </el-table-column>
            <el-table-column
                    prop="tuihuoTime"
                    header-align="center"
                    align="center"
                    label="退货时间">
            </el-table-column>
            <el-table-column
                    header-align="center"
                    align="center"
                    label="状态">
                <template slot-scope="scope">
                    <el-tag type="success" v-if="scope.row.status===0">未审核</el-tag>
                    <el-tag type="success" v-else-if="scope.row.status===1">已通过</el-tag>
                    <el-tag type="danger" v-else>被驳回</el-tag>
                </template>
            </el-table-column>

            <el-table-column
                    fixed="right"
                    header-align="center"
                    align="center"
                    width="150"
                    label="操作">
                <template slot-scope="scope">
                    <el-button v-if="scope.row.status===0" type="text" size="small" @click="agreHandle(scope.row.tuihuoId)">通过</el-button>
                    <el-button v-if="scope.row.status===0" type="text" size="small" @click="rejHandle(scope.row.tuihuoId)">驳回</el-button>
                    <el-button type="text" size="small" @click="detHandle(scope.row.tuihuoId)">查看详情</el-button>
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
        <pReturned v-if="pDetailedVisible" ref="pReturned" @refreshDataList="getDataList"></pReturned>
    </div>
</template>

<script>
    import pReturned from "../drugs_purchase/pReturned";

    export default {
        name: "all-returned",
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
                dataListLoading: false,
                dataListSelections: [],
                addOrUpdateVisible: false
            }
        },
        activated() {
            this.getDataList()
        },
        components: {
            pReturned
        },
        methods: {
            // 获取数据列表
            getDataList() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/allreturned'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'purchaseId': this.dataForm.key
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
            //查看详细
            detHandle(id) {
                this.pDetailedVisible = true
                this.$nextTick(() => {
                    this.$refs.pReturned.init(id)
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
            agreHandle(id){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/agreHandleReturned'),
                    method: 'post',
                    params: this.$http.adornParams({'tuihuoId': id})
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.$message.success("操作成功")
                        this.getDataList()
                    }else {
                        this.$message.error(data.msg)
                    }
                    this.dataListLoading = false
                })
            },
            rejHandle(id){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/rejHandleReturned'),
                    method: 'post',
                    params: this.$http.adornParams({'tuihuoId': id})
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.$message.success("操作成功")
                        this.getDataList()
                    }else {
                        this.$message.error(data.msg)
                    }
                    this.dataListLoading = false
                })
            }
        }
    }
</script>

<style scoped>

</style>
