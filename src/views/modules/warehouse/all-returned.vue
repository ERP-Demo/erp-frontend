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
                    <el-popover v-if="scope.row.checkName" trigger="hover" placement="top" @show="getReason(scope.row)">
                        <div v-loading="reasonLoading">
                            <p>操作人: {{ scope.row.checkName }}</p>
                            <p>驳回原因: {{ scope.row.reason }}</p>
                        </div>
                        <div slot="reference">
                            <el-tag type="danger" size="medium">被驳回</el-tag>
                        </div>
                    </el-popover>
                    <el-tag type="success" v-else>未提交</el-tag>
                </template>
            </el-table-column>

            <el-table-column
                    fixed="right"
                    header-align="center"
                    align="center"
                    width="150"
                    label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="subHandle(scope.row.processInstanceId)">提交</el-button>
                    <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.purchaseId)">修改</el-button>
                    <el-button type="text" size="small" @click="delHandle(scope.row.purchaseId)">删除</el-button>
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
            }
        }
    }
</script>

<style scoped>

</style>