<template>
    <!-- 化验
    模板管理 -->
    <div class="div1">
        <el-container>
            <!--  -->
            <transition name="el-zoom-in-left">

                <el-aside :width="asidewidth" style="margin-top:0;background:white;padding: 0 0 0 0">
                    <aside style="height:50px;margin:0 0 0 0">
                        <el-tag size="large">化验模板</el-tag>

                    </aside>
                    <div style="padding: 0 10px 0 10px">
                        <el-card v-if="!isaside">
                            <el-form style="background:white" :model="listQuery" inline>
                                <el-form-item label="模板名称:" style="width:280px">
                                    <el-input v-model="listQuery.name" placeholder="模板名称"></el-input>
                                </el-form-item>
                                <el-form-item label="范围:" style="width:280px">
                                    <el-select placeholder="请选择范围" v-model="listQuery.scope" clearable
                                               style="width: 130px" class="filter-item">
                                        <el-option
                                                v-for="item in [{key:0,value:'个人'},{key:1,value:'科室'},{key:2,value:'全院'}]"
                                                :key="item.key" :label="item.value" :value="item.key"/>
                                    </el-select>
                                </el-form-item>
                                <el-form-item style="width:400px">
                                    <el-button type="primary" @click="searchModel" v-if="!isaside">搜索模板</el-button>
                                    <el-button type="primary" @click="showaside('add')" v-if="!isaside">新建模板</el-button>
                                </el-form-item>
                            </el-form>
                        </el-card>
                        <div style="background:white;">
                            <el-table style="margin-bottom:50px" :data="dataList1">
                                <el-table-column align="center" label="模板编号" prop="testModelId" width="80">
                                    <template slot-scope="scope">
                                        {{scope.row.testModelId}}
                                    </template>
                                </el-table-column>
                                <el-table-column align="center" label="模板名称" prop="testModelName">
                                    <template slot-scope="scope">
                                        {{scope.row.testModelName}}
                                    </template>
                                </el-table-column>
                                <el-table-column align="center" label="模板简介" prop="testModelIntroduction">
                                    <template slot-scope="scope">
                                        {{scope.row.testModelIntroduction}}
                                    </template>
                                </el-table-column>
                                <el-table-column align="center" label="模版编码" prop="testModelCode">
                                    <template slot-scope="scope">
                                        {{scope.row.testModelCode}}
                                    </template>
                                </el-table-column>
                                <el-table-column align="center" label="范围" prop="testModelRange" width="80">
                                    <template slot-scope="scope">
                                        <span v-if="scope.row.testModelRange===0">个人</span>
                                        <span v-if="scope.row.testModelRange===1">科室</span>
                                        <span v-if="scope.row.testModelRange===2">全院</span>
                                    </template>
                                </el-table-column>
                                <el-table-column label="操作" prop="id" width="150px">
                                    <template slot-scope="scope">
                                        <el-button type="primary" size="mini" @click="editModel(scope.row)">修改
                                        </el-button>
                                        <el-button type="danger" size="mini" @click="deleteModel(scope.row)">删除
                                        </el-button>
                                    </template>
                                </el-table-column>
                            </el-table>
                            <pagination v-show="total>0" :total="total" :page.sync="listQuery.pageNum"
                                        :limit.sync="listQuery.pageSize" @pagination="getModelList"/>
                        </div>
                    </div>
                </el-aside>
            </transition>
            <!-- 编辑模板 -->
            <el-main style="padding:0 0 0 0;" v-if="isaside">
                <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="edit"
                           @click="updateModel">提交修改
                </el-button>
                <el-button type="primary" style="margin-left:30px;margin-top:30px;margin-bottom:30px" v-if="!edit"
                           @click="createModel">新建模板
                </el-button>
                <el-button type="danger" @click="showaside">取消</el-button>
                <el-form :model="datafrom" label-width="140px" inline>
                    <el-form-item label="模板名称">
                        <el-input placeholder="请输入模板名称" v-model="datafrom.testModelName" style="width:300px"></el-input>
                    </el-form-item>
                    <el-form-item label="模板简介">
                        <el-input placeholder="模板简介" v-model="datafrom.testModelIntroduction" style="width:300px"></el-input>
                    </el-form-item>
                    <el-form-item v-if='edit' label="模板编码">
                        <el-input placeholder="模板编码" v-model="datafrom.testModelCode" disabled style="width:300px"></el-input>
                    </el-form-item>
                    <el-form-item label="范围:" style="width:280px">
                        <el-select placeholder="请选择范围" v-model="datafrom.testModelRange" clearable style="width: 130px"
                                   class="filter-item">
                            <el-option v-for="item in [{key:0,value:'个人'},{key:1,value:'科室'},{key:2,value:'全院'}]"
                                       :key="item.key" :label="item.value" :value="item.key"/>
                        </el-select>
                    </el-form-item>
                </el-form>
                <el-button type="primary" style="margin-bottom:10px;margin-left:10px" @click="addItem">编辑项目</el-button>
                <el-table style="margin-bottom:50px" :data="dataList3">
                    <el-table-column label="化验项目id" prop="testSynthesizeId" width="550"></el-table-column>
                    <el-table-column label="化验项目名" prop="testSynthesizeName" width="550"></el-table-column>
                    <el-table-column label="化验项目价格(元)" prop="testSynthesizePrice" width="515"></el-table-column>
                </el-table>
            </el-main>
        </el-container>
        <el-dialog title="增加化验项目" :visible.sync="dialogVisible" width="650px" top="10px">
            <el-container>
                <!--        <el-aside width="30%" style="padding:0 0 0 0;margin:0 0 0 0">-->
                <!--            <el-tag type="primary" style="width:100%;height:40px;line-height: 40px;margin-top: 2%">药品目录-->
                <!--            </el-tag>-->
                <!--            <pagination layout="prev, pager, next" auto-scroll="false" style="margin-top:0px" page-sizes="[]"  v-show="total2>0" :total="total2" :page.sync="page.pageNum" :limit.sync="page.pageSize" @pagination="getdrugList(0)" />-->
                <!--          <div>-->
                <!--          </div>-->
                <!--        </el-aside>-->
                <el-main>
                    <el-button type="primary" style="float:right;margin-right: 6%" @click="addTest">确定</el-button>
                    <el-table :data="dataList2" cell-style="text-align:center" header-cell-style="text-align:center" style="margin-bottom: 5%" @selection-change="handleSelectionChange">
                        <el-table-column type="selection" header-align="center" align="center" width="50">
                        </el-table-column>
                        <el-table-column property="testSynthesizeId" label="化验项目id" width="150"></el-table-column>
                        <el-table-column property="testSynthesizeName" label="化验项目名" width="150"></el-table-column>
                        <el-table-column property="testSynthesizePrice" label="化验项目价格" width="200"></el-table-column>
                    </el-table>
                </el-main>
            </el-container>
        </el-dialog>
    </div>
</template>
<script>
    import {getNondrugModelList, updateModel, createModel, deleteModel} from '@/api/nondrugmodel'
    import {getAllNondrug} from '@/api/non_drug'
    import Pagination from '@/components/Pagination'
    import {deepClone, parseTime} from '@/utils'

    export default {
        components: {Pagination},
        data() {
            return {
                dataList1: [],
                dataList2: [],
                dataList3: [],
                page: {
                    pageNum: 1,
                    pageSize: 10,
                },
                oneprescription: {
                    name: '',
                    druglist: [],
                    amount: 0,
                    status: 0,
                },
                datafrom: {
                    testModelId:'',
                    testModelName: '',
                    testModelIntroduction: '',
                    testModelCode: '',
                    testModelRange: '',
                },
                itemdrugList: [],
                drugList: [],
                searchdrug: '',
                choices: [],
                dialogVisible: false,
                alldrugList: [],
                nondrugList: [],
                edit: 0,
                model: {},
                total2: 0,
                loading: false,
                isaside: false,
                asidewidth: "100%",
                total: 0,
                listQuery: {
                    id: '',
                    status: 1,
                    name: '',
                    scope: 0,
                    ownId: this.$store.getters.id,
                    aim: '',
                    code: '',
                    type: 1,
                    pageSize: 20,
                    pageNum: 1,
                    isAdmin: '0'
                },
                modelList: [],
                queryModel: {
                    name: '',
                    range: '',
                    type: ''
                }
            }
        },
        created() {
            this.getModelList()
        },
        activated() {
            this.getDataList()
            this.getDataList2()
        },
        methods: {
            // 获取数据列表
            getDataList() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/test_model/model/list'),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.dataList1 = data.page.list
                        // console.log("数据"+data.list)
                    } else {
                        this.dataList1 = []
                    }
                    this.dataListLoading = false
                })
            },
            getDataList2() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/test_synthesize/synthesize/list'),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.dataList2 = data.page.list
                        // console.log("数据"+data.list)
                    } else {
                        this.dataList2 = []
                    }
                    this.dataListLoading = false
                })

            },
          //获取复选框
          handleSelectionChange(val) {
            this.multipleSelection = val;
          },
            addTest() {
                this.dataList3=this.multipleSelection
                this.dialogVisible = false
            },
            async getdrugList(type) {
                let data = {}
                data.pageSize = this.page.pageSize
                data.pageNum = this.page.pageNum
                if (type !== 0)
                    data.typeId = type
                data.name = this.searchdrug
                const response = await getdrugList(data)
                this.drugList = response.data.list
                this.total2 = response.data.total
            },
            deldrug(row) {
                this.oneprescription.druglist = this.oneprescription.druglist.filter(item => {
                    if (item.id === row.id)
                        return false
                    return true
                })
                this.oneprescription.amount = 0
                this.oneprescription.druglist.forEach(item => {
                    this.oneprescription.amount += item.price
                })
                this.oneprescription.amount = Math.floor((this.oneprescription.amount + 0.5) * 100) / 100
            },
            choosedrug(val) {
                let flag = 1
                this.oneprescription.druglist.forEach(item => {
                    if (item.id === val.id) {
                        item.num += 1
                        flag = 0
                    }
                })
                if (flag) {
                    this.oneprescription.amount += val.price
                    this.oneprescription.amount = Math.floor((this.oneprescription.amount + 0.5) * 100) / 100
                    this.oneprescription.druglist.push(val)
                    this.oneprescription.druglist.forEach(item => {
                        if (item.num === undefined)
                            item.num = 1
                    })
                }
            },
            deleteModel(row) {
                this.$confirm('确认删除当前模板?', '警告', {
                    confirmButtonText: '确认',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    deleteModel(row.id).then(res => {
                        this.getModelList()
                        this.$notify({
                            title: '成功',
                            message: res.message,
                            type: 'success',
                            duration: 2000
                        })
                    })
                })
            },
            createModel() {
                var datafrom=this.datafrom
                console.log(this.datafrom);
                var ids = this.multipleSelection.map(item => {
                    return item.testSynthesizeId
                })
                this.$http({
                    url: this.$http.adornUrl(`/test_model/model/save`),
                    method: 'post',
                    data: this.$http.adornData({'testModel': datafrom, 'ids': ids},)

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
                                this.getDataList1()
                            }
                        })
                        console.log(this.datafrom)
                    } else {
                        this.$message.error(data.msg)
                    }
                })

                this.showaside()
            },
            updateModel() {
                let data = deepClone(this.model)
                data.nonDrugIdList = deepClone(this.choices)
                data.createTime = ''
                data.ownId = this.$store.getters.id
                updateModel(data).then(res => {
                    this.$notify({
                        title: '成功',
                        message: '修改成功',
                        type: 'success',
                        duration: 2000
                    })
                    this.getModelList()
                })
                this.showaside()
            },
            confirmItem() {
                this.nondrugList = this.alldrugList.filter(item => {
                    if (this.choices.includes(item.id))
                        return true
                })
                this.dialogVisible = !this.dialogVisible
            },
            addItem() {
                this.dialogVisible = true
                this.getdrugList()
            },
            getAllNondrug() {
                getAllNondrug().then(res => {
                    this.alldrugList = res.data
                    this.alldrugList.forEach(item => {
                        item.label = item.name
                        item.key = item.id
                    })
                })
            },
            async getModelList() {
                listModel(this.listQuery).then(res => {
                    this.modelList = res.data.list
                    this.modelList.forEach(model => {
                        model.dmsMedicineModelItemList.forEach(item => {
                            selectById(item.id).then(res => {
                                item.name = res.data.name
                                item.format = res.data.format
                                item.price = res.data.price
                                item.totalprice = item.price * item.num
                                item.mnemonicCode = res.data.mnemonicCode
                            })
                        })
                        model.createTime = parseTime(model.createTime)
                    })
                    this.total = res.data.total
                })

            },
            searchModel() {
                this.loading = true
                this.getModelList()
                this.loading = false
            },
            showaside(type) {
                if (this.asidewidth === "100%")
                    this.asidewidth = "0%"
                else
                    this.asidewidth = "100%"
                this.isaside = !this.isaside
                if (type === 'edit')
                    this.edit = 1
                else if (type === 'add') {
                    this.choices = []
                    this.nondrugList = []
                    this.itemdrugList = []
                    this.model = {}
                    this.edit = 0
                } else
                    this.edit = 0
            },
            async editModel(row) {
                this.model = deepClone(row)
                this.itemdrugList = this.model.dmsMedicineModelItemList
                this.showaside('edit')
            }
        }
    }
</script>
<style>
    body .el-table th.gutter {
        display: table-cell !important;
    }

    .el-transfer-panel {
        width: 300px;
    }
</style>
