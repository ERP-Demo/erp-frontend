<template>
    <div class="mod-config">
        <el-form :inline="true">
            <el-button type="primary" @click="addPur=true,addHandle()">新增</el-button>
        </el-form>
        <el-dialog title="添加订单"
                :visible.sync="addPur">
            <el-form :rules="dataRule" @keyup.enter.native="dataFormSubmit()"
                     label-width="120px">
                <el-form-item label="供货商" style="display: inline-block;" prop="supplierId">
                    <el-select v-model="sname.supplierId" filterable placeholder="请选择供货商">
                        <el-option
                                v-for="item in sname"
                                :key="item.supplierId"
                                :label="item.supplierName"
                                :value="item.supplierId">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item style="display: inline-block;" label="新增商品" prop="DrugsDetailed">
                    <el-select v-model="did" filterable placeholder="请选择商品">
                        <el-option
                                v-for="item in dname"
                                :key="item.drugsId"
                                :label="item.drugsName"
                                :value="item.drugsId">
                        </el-option>
                    </el-select>
                    <el-button type="primary" @click="addDetailed()">新增</el-button>
                </el-form-item>
                <el-table
                        :data="detailed"
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
                            <el-input-number v-model="scope.row.price" :precision="2" :min="0.01"
                                             :step="0.1"></el-input-number>
                        </template>
                    </el-table-column>
                    <el-table-column
                            header-align="center"
                            align="center"
                            label="进货数量">
                        <template slot-scope="scope">
                            <el-input-number v-model="scope.row.num" :min="1" :precision="0"></el-input-number>
                        </template>
                    </el-table-column>
                </el-table>
            </el-form>
            <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()" :disabled="confirmButtonDisabled">确定</el-button>
    </span>
        </el-dialog>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                addPur: false,
                did:'',
                sname:[],
                dname:[],
                detailed:[],
                dataRule: {
                    supplierId: [{ required: true, message: '供货商不能为空', trigger: 'change' }],
                    DrugsDetailed: [{ required: true, message: '商品不能为空', trigger: 'change' }]
                }
            }
        },
        methods: {
            addHandle(){
                this.getSname(),
                this.getDname()
            },
            getSname(){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/sname'),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.sname = data.sname
                    } else {
                        this.sname = []
                    }
                    this.dataListLoading = false
                })
            },
            getDname(){
                this.$http({
                    url: this.$http.adornUrl('/drugs_purchase/purchase/dname'),
                    method: 'get'
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.dname = data.dname
                    } else {
                        this.dname = []
                    }
                    this.dataListLoading = false
                })
            },
            dataFormSubmit () {
                this.$refs['dataForm'].validate((valid) => {
                    if (valid) {
                        this.$http({
                            url: this.$http.adornUrl(`/admin/supplier/orders/${!this.dataForm.id ? 'save' : 'update'}`),
                            method: !this.dataForm.id ? 'post' : 'put',
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
                            } else {
                                this.$message.error(data.msg)
                            }
                        })
                    }
                })
            },
            addDetailed () {
                let flag = false
                this.detailed.map(item => {
                    if (item.id === this.did) {
                        item.num++
                        flag = true
                        return item
                    }
                })
                if (flag) return
                this.detailed.forEach(item => {
                    if (item.id === this.did) {
                        this.$set(item, 'num', 1)
                        this.$set(item, 'price', 0.01)
                        this.detailed.push(item)
                    }
                })
            }
        }
    }
</script>