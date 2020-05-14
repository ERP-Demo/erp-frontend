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
                    label="进货单价">
            </el-table-column>
            <el-table-column
                    header-align="center"
                    align="center"
                    prop="num"
                    label="进货数量">
            </el-table-column>
            <el-table-column
                    header-align="center"
                    align="center"
                    label="退货数量">
                <template slot-scope="scope">
                    <el-input-number v-model="scope.row.pdNum" :min="0" :max="scope.row.num" :precision="0"></el-input-number>
                </template>
            </el-table-column>
        </el-table>

        <span slot="footer" class="dialog-footer">
            <!--<div>
                <p>退货金额：<el-span prop="totalPrice">{{ totalPrice }}</el-span></p>
            </div>-->
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">退货</el-button>
        </span>

    </el-dialog>
</template>

<script>
    export default {
        data() {
            return {
                dataForm: {
                    purchaseReturned: [],
                },
                visible: false,
                tableData: [],
                totalPrice: 0,
                purchaseId: ''
            }
        },
        activated() {
            this.init()
        },
        methods: {
            init(id) {
                this.visible=true
                this.purchaseId=id
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/all/'+id),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        data.all.map(item => {
                            this.tableData.push({
                                'pdid': item.pdid,
                                'drugsName':item.drugsName,
                                'pdMoney': item.pdMoney,
                                'num': item.pdNum,
                                'pdNum': 0
                            })
                        })
                    } else {
                        this.tableData = []
                    }
                    this.dataListLoading = false
                })
            },
            dataFormSubmit(){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/purchaseReturned'),
                    method: 'post',
                    data: this.$http.adornData({'purchaseReturned':{'purchaseId':this.purchaseId},'purchaseReturnedDetaileds':this.tableData})
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
                        this.$router.replace({ name: 'warehouse-all-returned' })
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            }
        }
    }
</script>
