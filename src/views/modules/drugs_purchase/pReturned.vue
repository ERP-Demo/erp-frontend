<template>
    <el-dialog
            title="查看详情"
            :visible.sync="visible"
            width="66%">
        <el-table
                ref="multipleTable"
                :data="tableData"
                tooltip-effect="dark"
                style="width: 100%">查看药品
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
                    label="退货数量">
            </el-table-column>
        </el-table>
    </el-dialog>
</template>

<script>
    export default {
        name: "pReturned",
        data() {
            return {
                visible: false,
                tableData: []
            }
        },
        methods: {
            init(id) {
                this.visible=true
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/selectReturned/'+id),
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
        }
    }
</script>

<style scoped>

</style>