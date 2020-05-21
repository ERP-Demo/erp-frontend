<template>
    <div class="mod-config">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.name" placeholder="患者名称" clearable></el-input>
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
                    prop="testSynthesizeId"
                    header-align="center"
                    align="center"
                    label="序号">
            </el-table-column>
            <el-table-column
                    prop="patientName"
                    header-align="center"
                    align="center"
                    label="患者名称">
            </el-table-column>
            <el-table-column
                    prop="purpose"
                    header-align="center"
                    align="center"
                    label="目的">
            </el-table-column>
            <el-table-column
                    prop="requirements"
                    header-align="center"
                    align="center"
                    label="要求">
            </el-table-column>
            <el-table-column
                    prop="clinicalImpression"
                    header-align="center"
                    align="center"
                    label="临床印象">
            </el-table-column>
            <el-table-column
                    prop="clinicalDiagnosis"
                    header-align="center"
                    align="center"
                    label="临床诊断">
            </el-table-column>
            <el-table-column
                    prop="checkThe"
                    header-align="center"
                    align="center"
                    label="检查部位">
            </el-table-column>
            <el-table-column
                    fixed="right"
                    header-align="center"
                    align="center"
                    width="150"
                    label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="showReqChilren(scope.row.testSynthesizeId)">详细</el-button>
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
        <chilren v-if="childrenVisible" ref="chilren" @refreshDataList="getDataList"></chilren>
    </div>
</template>

<script>
    import chilren from './testSheetChilren'
    export default {
        name: "test-sheet",
        data () {
            return {
                dataForm: {
                    name: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataListLoading: false,
                dataListSelections: [],
                addOrUpdateVisible: false,
                childrenVisible: false
            }
        },
        activated () {
            this.getDataList()
        },
        components: {
            chilren
        },
        methods: {
            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/requirements/testsheet/selectTestSheet'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'patientName': this.dataForm.name
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
            showReqChilren(id){
                this.childrenVisible = true
                this.$nextTick(() => {
                    this.$refs.chilren.init(id)
                })
            },
            // 每页数
            sizeChangeHandle (val) {
                this.pageSize = val
                this.pageIndex = 1
                this.getDataList()
            },
            // 当前页
            currentChangeHandle (val) {
                this.pageIndex = val
                this.getDataList()
            },
            // 多选
            selectionChangeHandle (val) {
                this.dataListSelections = val
            }
        }
    }
</script>

<style scoped>

</style>