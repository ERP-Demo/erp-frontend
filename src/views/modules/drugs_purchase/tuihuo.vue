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
                    @change="count"
                    label="进货单价">
            </el-table-column>
            <el-table-column
                    header-align="center"
                    align="center"
                    label="进货数量">
                <template slot-scope="scope">
                    <el-input-number prop="pdNum" @change="count"  v-model="scope.row.pdNum" :min="0" :precision="0"></el-input-number>
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
            }
        },
        activated() {
            this.init()
        },
        methods: {
            init(id) {
                let ids=this.$route.params.id
                this.visible=true
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/all/'+id),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.tableData = data.all
                        console.log(this.tableData)
                    } else {
                        this.tableData = []
                    }
                    this.dataListLoading = false
                })
            },
            //计算总价格
            count(value){
                this.totalPrice=0
                for (const d of this.tableData) {
                    this.totalPrice=this.NumberAdd(this.totalPrice,this.NumberMul(d.pdNum,d.pdMoney));
                }
            },
            NumberMul(arg1, arg2) {
                var m = 0;
                var s1 = arg1.toString();
                var s2 = arg2.toString();
                try {
                    m += s1.split(".")[1].length;
                } catch (e) {}
                try {
                    m += s2.split(".")[1].length;
                } catch (e) {}

                return Number(s1.replace(".", "")) * Number(s2.replace(".", "")) / Math.pow(10, m);
            },
            NumberAdd(arg1, arg2) {
                var r1, r2, m, n;
                try {
                    r1 = arg1.toString().split(".")[1].length
                } catch (e) {
                    r1 = 0
                }
                try {
                    r2 = arg2.toString().split(".")[1].length
                } catch (e) {
                    r2 = 0
                }
                m = Math.pow(10, Math.max(r1, r2))
                n = (r1 >= r2) ? r1 : r2;
                return ((arg1 * m + arg2 * m) / m).toFixed(n);
            },
            dataFormSubmit(){
                this.dataForm.purchaseReturned=this.tableData;
                console.log(this.tableData)
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/purchaseReturned'),
                    method: 'post',
                    data: this.$http.adornData(this.dataForm)
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