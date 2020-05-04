<template>
    <div class="mod-config">
        <el-form :rules="dataRule" @keyup.enter.native="dataFormSubmit()"
                 label-width="120px">
            <el-form-item label="供货商：" style="display: inline-block;" prop="dataForm.supplierId">
                <el-select v-model="dataForm.supplierId" filterabdataFormSubmitle placeholder="请选择供货商" @blur="btnDrugs">
                    <el-option
                            v-for="item in supplier"
                            :key="item.supplierId"
                            :label="item.supplierName"
                            :value="item.supplierId">
                    </el-option>
                </el-select>
            </el-form-item>

            <el-button type="primary" @click="dialogVisible1=true,selectSupplierIdByDrugs()" :disabled="(btnBoolean == btnStatus)== 1 ? true : false">查看药品</el-button>

            <el-table
                    :data="dataForm.detailed"
                    border
                    style="width: 100%;">
                <el-table-column
                        prop="drugsName"
                        header-align="center"
                        align="center"
                        label="商品名称">
                </el-table-column>
                <el-table-column
                        header-align="center"
                        align="center"
                        label="进货单价">
                    <template slot-scope="scope">
                        <el-input-number prop="pdMoney" @change="count" v-model="scope.row.price" :precision="2" :min="0.01" :step="0.1"></el-input-number>
                    </template>
                </el-table-column>
                <el-table-column
                        header-align="center"
                        align="center"
                        label="进货数量">
                    <template slot-scope="scope">
                        <el-input-number prop="pdNum" @change="count"  v-model="scope.row.num" :min="1" :precision="0"></el-input-number>
                    </template>
                </el-table-column>
                <el-table-column
                        fixed="right"
                        header-align="center"
                        align="center"
                        width="160"
                        label="操作">
                    <template slot-scope="scope">
                        <el-button type="text" size="small" @click="removeList(scope.$index, scope.row)">移除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-form>

        <span slot="footer" class="dialog-footer" style="float: right;margin-top: 10px;">
            <div>
                <p>应付金额：<el-span prop="totalPrice">{{ totalPrice }}</el-span></p>
                <p>已付订金：<el-input v-model="dataForm.payPrice" placeholder="数量" style="width: 250px;line-height: 50px"></el-input></p>
            </div>
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        </span>

        <el-dialog
                title="提示"
                :visible.sync="dialogVisible1"
                width="66%"
                :before-close="handleClose">
            <el-table
                    ref="multipleTable"
                    :data="drugsList"
                    tooltip-effect="dark"
                    style="width: 100%;"
                    v-loading="dataListLoading"
                    @selection-change="selectionChangeHandle">
                <el-table-column
                        type="selection"
                        width="55">
                </el-table-column>
                <el-table-column
                        prop="drugsId"
                        header-align="center"
                        align="center"
                        label="药品编号">
                </el-table-column>
                <el-table-column
                        prop="drugsName"
                        header-align="center"
                        align="center"
                        label="药品名称" >
                </el-table-column>
                <el-table-column
                        prop="drugsPrice"
                        header-align="center"
                        align="center"
                        label="药品单价">
                </el-table-column>
                <el-table-column
                        prop="drugsNorms"
                        header-align="center"
                        align="center"
                        label="药品规格">
                </el-table-column>
                <el-table-column
                        prop="drugsUsage"
                        header-align="center"
                        align="center"
                        label="药品用法">
                </el-table-column>
                <el-table-column
                        prop="drugsDosage"
                        header-align="center"
                        align="center"
                        label="药品用量">
                </el-table-column>
                <el-table-column
                        prop="drugsTaboo"
                        header-align="center"
                        align="center"
                        label="禁忌">
                </el-table-column>
                <el-table-column
                        prop="drugsProducer"
                        header-align="center"
                        align="center"
                        label="生产厂商">
                </el-table-column>
                <el-table-column
                        prop="drugsApprovalNumber"
                        header-align="center"
                        align="center"
                        label="批准文号">
                </el-table-column>
            </el-table>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible1 = false">取 消</el-button>
                <el-button type="primary" @click="addDrugs()">确 定</el-button>
            </span>

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
<!--            <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>-->
        </el-dialog>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                dataForm: {
                    detailed:[],
                    payPrice:0,
                    supplierId:''
                },
                totalPrice: 0,
                visible: false,
                confirmButtonDisabled: false,
                dataListLoading: false,
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                btnBoolean: -1,
                btnStatus: -1,
                did:'',
                supplier:[],
                drugs:[],
                drugsList:[],
                price: '',
                num:'',
                multipleSelection: [],
                DrugsAndDetailedList:[],
                dialogVisible1: false,
                dataRule: {
                    supplier: [{ required: true, message: '供货商不能为空', trigger: 'change' }],
                    drugs: [{ required: true, message: '药品不能为空', trigger: 'change' }]
                }
            }
        },
        activated() {
            this.getSupplier()
            this.getDrugs()
        },
        methods: {
            //获取供应商（下拉列表）
            getSupplier(){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/supplier'),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.supplier = data.supplier
                    } else {
                        this.supplier = []
                    }
                    this.dataListLoading = false
                })
            },
            //获取药品（下拉列表）
            getDrugs(){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/drugs'),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.drugs = data.drugs
                    } else {
                        this.drugs = []
                    }
                    this.dataListLoading = false
                })
                this.dataForm.drugs = drugs || []
                this.dataForm.drugs.forEach(item => {
                    this.$set(item, 'num', 1)
                    this.$set(item, 'price', 0.01)
                })
            },
            //根据供应商解除按钮禁用
            btnDrugs(){
                if(this.dataForm.supplierId.selected != ''){
                    this.btnBoolean=1
                    this.btnStatus=-1
                }
            },
            //根据供应商查询对应的药品
            selectSupplierIdByDrugs(){
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/SupplierAndDrugs/selectSupplierIdByDrugs'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'supplierId': this.dataForm.supplierId
                    })
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.drugsList = data.page.list
                        this.totalPage = data.page.totalCount
                    } else {
                        this.dataList = []
                        this.totalPage = 0
                    }
                    this.dataListLoading = false
                })
            },
            //添加药品到前端页面
            addDrugs () {
                this.multipleSelection.map(item =>{
                    this.dataForm.detailed.push({
                        'price':1,
                        'num':1,
                        'drugsId':item.drugsId,
                        'drugsName':item.drugsName
                    })
                })
                this.count()
                this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1000,
                    onClose: () => {
                        this.dialogVisible1 = false
                        this.$emit('refreshDataList')
                    }
                })
            },
            removeList(index){ //删除行数
                this.dataForm.detailed.splice(index, 1)
            },
            //计算总价格
            count(value){
                this.totalPrice=0
                for (const d of this.dataForm.detailed) {
                    this.totalPrice=this.NumberAdd(this.totalPrice,this.NumberMul(d.num,d.price));
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
            //添加
            dataFormSubmit () {
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/addSupplierAndDrugs'),
                    method: 'post',
                    data: this.$http.adornData(this.dataForm,true)
                }).then(({data}) => {
                    this.confirmButtonDisabled = true
                    if (data && data.code === 200) {
                        this.$message({
                            message: '操作成功',
                            type: 'success',
                            duration: 1000
                        })
                        /*刷新页面*/
                        this.dataForm.detailed=[]
                        this.dataForm.payPrice=0
                        this.dataForm.supplierId='
                        this.totalPrice=''
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            },
            // 每页数
            sizeChangeHandle (val) {
                this.pageSize = val
                this.pageIndex = 1
                this.getDataList()
            },
            //关闭
            handleClose(done) {
                done();
            },
            // 当前页
            currentChangeHandle (val) {
                this.pageIndex = val
                this.getDataList()
            },
            // 多选
            selectionChangeHandle (val) {
                this.dataListSelections = val
            },
            //获取复选框
            selectionChangeHandle(val) {
                this.multipleSelection = val;
            }
        }
    }
</script>
<style>
    /*.el-table__row td .cell{*/
    /*    white-space:nowrap;*/
    /*    overflow:hidden;*/
    /*    text-overflow:ellipsis;*/
    /*}*/
</style>
