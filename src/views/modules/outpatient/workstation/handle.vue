<template>
    <!-- 处置申请 -->
    <el-container>
        <el-aside :width="mainwidth" style="background:white;padding:0 10px 0 0">
            <aside style="margin:0 0 0 0">
                <el-button type="text" size="medium" @click="addcheck"><i class="el-icon-circle-plus-outline"/>新增项目
                </el-button>
                <el-button type="text" size="medium" @click="delcheck"><i class="el-icon-remove-outline"/>删除项目
                </el-button>
                <el-button type="text" size="medium" @click="apply"><i class="el-icon-circle-check"/>开立项目</el-button>
                <el-button type="text" size="medium" @click="invalid"><i class="el-icon-circle-close"/>作废项目</el-button>
                <el-button type="text" size="medium" @click="saveNonDrug"><i class="el-icon-upload2"/>暂存</el-button>
                <el-button type="text" size="medium" @click="getNonDrug"><i class="el-icon-download"/>取出暂存项</el-button>
                <el-button type="text" size="mini" @click="refresh"><i class="el-icon-refresh"/>刷新</el-button>
                <el-button style="float:right" @click="controlfast"><i v-show="!isclose" class="el-icon-caret-right"/><i
                        v-show="isclose" class="el-icon-caret-left"/></el-button>
            </aside>
            <el-tag type="primary">项目金额总计:</el-tag>
            <el-tag type="warning">{{totalprice}} 元</el-tag>
            <el-table
                    ref="multipleTable"
                    :data="record"
                    border
                    tooltip-effect="dark"
                    style="width: 100%"
                    @selection-change="handleSelectionChange">
                <el-table-column
                        align="center"
                        type="selection"
                        width="55"
                        @selection-change="handleSelectionChange">
                </el-table-column>
                <el-table-column
                        align="center"
                        label="项目编码"
                        width="120">
                    <template slot-scope="scope">{{ scope.row.id }}</template>
                </el-table-column>
                <el-table-column
                        align="center"
                        prop="handleName"
                        label="项目名称"
                        width="200">
                </el-table-column>
                <el-table-column
                        align="center"
                        prop="handlePrice"
                        label="单价"
                        show-overflow-tooltip>
                </el-table-column>
                <el-table-column
                        align="center"
                        label="状态"
                        show-overflow-tooltip>
                    <template slot-scope="scope">
                        <el-tag type="warning" v-if="scope.row.status===-1">未开立</el-tag>
                        <el-tag type="danger" v-if="scope.row.status===0">已作废</el-tag>
                        <el-tag type="info" v-if="scope.row.status===1">未缴费</el-tag>
                        <el-tag type="info" v-if="scope.row.status===2">未登记</el-tag>
                        <el-tag type="info" v-if="scope.row.status===3">已登记</el-tag>
                        <el-tag type="success" v-if="scope.row.status===4">已执行</el-tag>
                    </template>
                </el-table-column>
            </el-table>
        </el-aside>
        <transition name="el-zoom-in-left">
            <el-main width="50%" v-show="isclose"
                     style="border-style: dotted;border-width: 0px 0px 0px 1px;border-color:#C0C0C0;margin-top:-12px">
                <el-tabs v-model="activeName" @tab-click="handleClick">
                    <el-tab-pane label="常用模板" name="first">
                        <el-table highlight-current-row @row-dblclick="addModel" @row-click="selectModel" stripe
                                  :data="checkmodels" height="230">
                            <el-table-column label="模板名">
                                <template slot-scope="scope">
                                    {{scope.row.name}}
                                </template>
                            </el-table-column>
                            <el-table-column label="模板简介">
                                <template slot-scope="scope">
                                    {{scope.row.aim}}
                                </template>
                            </el-table-column>
                        </el-table>
                        <el-card v-show="onemodel.name!==undefined" class="box-card" shadow="never"
                                 body-style="font-size: 14px;font-family:'微软雅黑';width:350px">
                            <div slot="header" class="clearfix">
                                <span>模板详情</span>
                            </div>
                            <p><b>模板名:</b> {{onemodel.name}}</p>
                            <p><b>模板内容:</b></p>
                            <p v-for="(nondrug,index) in onemodel.nondruglist" :key="index"><b></b> {{nondrug.name}}</p>
                            <p><b>总金额：</b>{{onemodel.totalprice}} 元</p>
                        </el-card>
                    </el-tab-pane>
                    <el-tab-pane label="常用处置项" name="second">
                        <el-table :data="freqlist" highlight-current-row @row-click="addfreitem">
                            <el-table-column label="项目名" prop="handleName"></el-table-column>
                            <el-table-column label="项目单价" prop="handlePrice"></el-table-column>
                            <el-table-column label="编码" prop="id"></el-table-column>
                        </el-table>
                    </el-tab-pane>
                </el-tabs>
            </el-main>
        </transition>
        <el-dialog title="检查目录" :visible.sync="dialogTableVisible" top="50px">
            <div style="height:450px">
                <span>搜索检查</span>
                <el-input style="width:200px;" size="mini" v-model="search" placeholder="搜索检查"></el-input>
                <el-button type="primary" size="mini" @click="getfreqList" style="width: 20px;margin-left: 2px"><icon-svg name="search" style="margin-left:-6px"/></el-button>
                <el-table height="350px" @row-click="selectCheck" :data="freqlist" highlight-current-row
                          style="margin-top:20px">
                    <el-table-column label="项目名" prop="handleName"></el-table-column>
                    <el-table-column label="项目单价" prop="handlePrice"></el-table-column>
                    <el-table-column label="编码" prop="id"></el-table-column>
                </el-table>
                <el-pagination
                        @size-change="sizeChangeHandle"
                        @current-change="currentChangeHandle"
                        :current-page="pageIndex"
                        :page-sizes="[10, 20, 50, 100]"
                        :page-size="pageSize"
                        :total="totalPage"
                        style="text-align: center"
                        layout="total, sizes, prev, pager, next, jumper">
                </el-pagination>
            </div>
        </el-dialog>
    </el-container>
</template>
<script>
    import {deepClone} from '@/utils'
    import {Promise} from 'q';
    export default {
        props: ['patient', 'registerId'],
        name: 'Inspection',
        data() {
            return {
                freqlist: [],
                onemodel: {},
                totalprice: 0.000,
                ref: [],
                checkmodels: [],
                check: {},
                demandVisible: false,
                dialogTableVisible: false,
                activeName: 'first',
                isclose: true,
                checkList: [],
                mainwidth: "65%",
                activeNames: ['1'],
                total: 0,
                search: '',
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                record: [],
                applyId: ''
            };
        },
        watch: {
            'patient': function (newVal) {
                this.patient = newVal
                this.listRecord()
            },
        },
        created() {
            Promise.all([
                this.getNondrugList().then(() => {
                    this.getmodel()
                })
            ])
            this.getfreqList()
        },
        methods: {
            saveNonDrug() {
                let data = {}
                data.dmsNonDrugItemRecordParamList = this.ref
                data.registrationId = this.patient.registrationId
                data.type = 2
                saveNonDrug(data).then(res => {
                    this.$notify({
                        title: '成功',
                        message: '已暂存选中的处置项',
                        type: 'success',
                        duration: 2000
                    })
                })
            },
            getNonDrug() {
                let data = {}
                data.registrationId = this.patient.registrationId
                data.type = 2
                getNonDrug(data).then(res => {
                    res.data.dmsNonDrugItemRecordParamList.forEach(item => {
                        this.selectCheckred(item)
                    })
                    this.$notify({
                        title: '成功',
                        message: '已取出暂存的处置项',
                        type: 'success',
                        duration: 2000
                    })
                })
            },
            addfreitem(val) {
                this.selectCheck(val)
            },
            getfreqList() {
                this.$http({
                    url: this.$http.adornUrl('/patient_handle/handle/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'search': this.search
                    })
                }).then(({data}) => {
                    if (data && data.code === 200) {
                        this.freqlist = data.page.list
                        this.totalPage = data.page.totalCount
                    } else {
                        this.dataList = []
                        this.totalPage = 0
                    }
                    this.dataListLoading = false
                })
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
            addModel(val) {
                this.$confirm('是否确定将 模板:' + val.name + ' 中的内容导入该患者的检查中', '导入模板', {
                    confirmButtonText: '确认',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    val.nondruglist.forEach(item => {
                        item.status = -1
                        let flag = 1
                        this.record.forEach(com => {
                            if (com.id === item.id) {
                                flag = 0
                            }
                        })
                        if (flag)
                            this.record.push(item)
                    })
                })
            },
            selectModel(val) {
                this.onemodel = val
                console.log(this.onemodel)
            },
            async getmodel() {
                let data = {}
                data.scope = 0
                data.ownId = this.$store.getters.id
                data.type = 2
                data.pageSize = 10000
                data.pageNum = 1
                data.isAdmin = 0
                await getNondrugModelList(data).then(res => {
                    this.checkmodels = res.data.list
                    this.checkmodels.forEach(onemodel => {
                        onemodel.nondruglist = []
                        onemodel.totalprice = 0.00
                        this.checkList.filter(item => {
                            if (onemodel.nonDrugIdList.includes(item.id)) {
                                onemodel.nondruglist.push(item)
                                onemodel.totalprice += item.price
                            }
                        })
                    })
                })
            },
            refresh() {
                this.$confirm('未开立的项目都将清除,确认刷新?', '刷新', {
                    confirmButtonText: '确认',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.listRecord()
                })
            },
            invalid() {
                let data = ''
                this.ref.forEach(item => {
                    data += (item.id + ',')
                })
                data = data.substr(0, data.length - 1)
                invalid(data).then(res => {
                    this.$notify({
                        title: '成功',
                        message: res.message,
                        type: 'success',
                        duration: 2000
                    })
                })
            },
            async listRecord() {
                this.$http({
                    url: this.$http.adornUrl(`/patient_handle/handle/apply/list/${this.registerId}`),
                    method: 'get'
                }).then(({data}) => {
                    if (data.code == 200) {
                        this.applyId=data.id
                        this.record=[]
                        data.list.map(item =>{
                            item.patientHandle.status=item.status
                            this.record.push(item.patientHandle)
                        })
                    }
                })
            },
            handleSelectionChange(val) {
                this.ref = val
                this.totalprice = 0.00
                this.ref.forEach(item => {
                    this.totalprice += item.handlePrice
                })
                this.totalprice = this.totalprice.toFixed(2)
            },
            apply() {
                let data = []
                let ids=[]
                this.ref.map(item =>{
                    if (item.status===-1) {
                        ids.push(item.id)
                        data.push({'handleId': item.id})
                    }
                })
                if (data.length===0){
                    this.$message.error("请选择未开立项目")
                    return
                }
                this.$http({
                    url: this.$http.adornUrl('/patient_handle/handle/apply'),
                    method: 'post',
                    data: this.$http.adornData({
                        'apply': {
                            'registerId': this.registerId,
                            'patientId': this.patient.patientId
                        }, 'detaileds': data
                    })
                }).then(({data}) => {
                    if (data.code == 200) {
                        let d=[]
                        this.record.map(item => {
                            if (ids.includes(item.id)) {
                                item.status=1
                                d.push(item)
                            }
                        })
                        this.record=d
                        this.applyId = data.id
                        this.$message.success("开立成功")
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            },
            submitDemand() {
                this.record.forEach(item => {
                    if (item.id === this.check.id) {
                        item.aim = this.check.aim
                        item.demand = this.check.demand
                        item.clinicalImpression = this.check.clinicalImpression
                        item.clinicalDiagnosis = this.check.clinicalDiagnosis
                        item.checkParts = this.check.checkParts
                    }
                })
                this.demandVisible = false
            },
            demand(row) {
                this.demandVisible = true
                this.check = deepClone(row)
            },
            async getNondrugList() {
                const response = await getNondrugList(this.listQuery)
                console.log(response)
                this.checkList = response.data.list
                this.total = response.data.total
            },
            selectCheckred(val) {
                let flag = 1
                val.status = -1
                this.record.forEach(item => {
                    if (item.id === val.id) {
                        flag = 0
                    }
                })
                if (flag)
                    this.record.push(val)
                this.dialogTableVisible = false
            },
            selectCheck(val) {
                val.status = 0
                this.$confirm('是否添加 ' + val.handleName + ' 到该患者?', '添加检查', {
                    confirmButtonText: '确认',
                    cancelButtonText: '取消',
                    type: 'success'
                }).then(() => {
                    let flag = 1
                    val.status = -1
                    this.record.forEach(item => {
                        if (item.id === val.id) {
                            flag = 0
                        }
                    })
                    if (flag)
                        this.record.push(val)
                    else
                        this.$message.error('已存在该检查项目！')
                    this.dialogTableVisible = false
                })
            },
            addcheck() {
                this.dialogTableVisible = true
            },
            delcheck() {
                let data = []
                this.record.map(item => {
                    if (!(item.status === -1 && this.ref.includes(item))) {
                        data.push(item)
                    }
                })
                this.record = data
            },
            controlfast() {
                this.isclose = !this.isclose
                if (this.mainwidth === "65%")
                    this.mainwidth = "80%"
                else
                    this.mainwidth = "65%"
            }
        }
    }
</script>