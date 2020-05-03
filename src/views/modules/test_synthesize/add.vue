<template>
    <div class="mod-config">
        <el-form :model="dataForm" :rules="dataRule" @keyup.enter.native="dataFormSubmit()" label-width="120px">
            <el-form-item label="化验名称" prop="synthesizeName">
                <el-input v-model="dataForm.synthesizeName" placeholder="化验名称" style="width: 200px;"></el-input>
            </el-form-item>
            <el-form-item label="化验价格" prop="synthesizPrice">
                <el-input v-model="dataForm.synthesizPrice" placeholder="化验价格" style="width: 200px;"></el-input>
            </el-form-item>

            <el-button type="primary" @click="dialogVisible1=true,getDataList()" style="margin: 0 0 15px 40px;">查看化验项目</el-button>

            <el-table
                    :data="dataForm.detailed"
                    border
                    style="width: 100%;">
                <el-table-column
                        type="selection"
                        header-align="center"
                        align="center"
                        width="50">
                </el-table-column>
                <el-table-column
                        prop="testProjectsId"
                        header-align="center"
                        align="center"
                        label="化验项目的编号">
                </el-table-column>
                <el-table-column
                        prop="testAbbreviation"
                        header-align="center"
                        align="center"
                        label="化验项目的全称">
                </el-table-column>
                <el-table-column
                        header-align="center"
                        align="center"
                        label="上限">
                    <template slot-scope="scope">
                        <el-input-number prop="pdMoney" v-model="scope.row.ceiling" :precision="0" :min="1" :step="1"></el-input-number>
                    </template>
                </el-table-column>
                <el-table-column
                        header-align="center"
                        align="center"
                        label="下限">
                    <template slot-scope="scope">
                        <el-input-number prop="pdMoney" v-model="scope.row.floor" :precision="0" :min="1" :step="1"></el-input-number>
                    </template>
                </el-table-column>
                <el-table-column
                        header-align="center"
                        align="center"
                        label="单位">
                    <template slot-scope="scope">
                        <el-input v-model="scope.row.unit" placeholder="单位" style="width: 105px"></el-input>
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
                    :data="dataList"
                    tooltip-effect="dark"
                    style="width: 100%;"
                    v-loading="dataListLoading"
                    @selection-change="selectionChangeHandle">
                <el-table-column
                        type="selection"
                        header-align="center"
                        align="center"
                        width="50">
                </el-table-column>
                <el-table-column
                        prop="testAbbreviation"
                        header-align="center"
                        align="center"
                        label="化验项目的简称">
                </el-table-column>
                <el-table-column
                        prop="testName"
                        header-align="center"
                        align="center"
                        label="化验项目的全称">
                </el-table-column>
            </el-table>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible1 = false">取 消</el-button>
                <el-button type="primary" @click="addSynthesize()" :disabled="confirmButtonDisabled">确 定</el-button>
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
                    synthesizeName: '',
                    synthesizPrice: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                visible: false,
                confirmButtonDisabled: false,
                dataListLoading: false,
                dataListSelections: [],
                addOrUpdateVisible: false,
                multipleSelection: [],
                dialogVisible1: false,
                dataRule: {
                    synthesizeName: [{ required: true, message: '化验名称不能为空', trigger: 'change' }],
                    synthesizPrice: [{ required: true, message: '化验价格不能为空', trigger: 'change' }]
                }
            }
        },
        activated() {
            this.getDataList()
        },
        methods: {
            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/test_projects/projects/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'key': this.dataForm.key
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
            //添加药品到前端页面
            addSynthesize () {
                this.multipleSelection.map(item =>{
                    this.dataForm.detailed.push({
                        'testProjectsId':item.testProjectsId,
                        'testAbbreviation':item.testAbbreviation,
                        'ceiling':1,
                        'floor':1,
                    })
                })
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
            //添加
            dataFormSubmit () {
                this.$http({
                    url: this.$http.adornUrl('/test_synthesize/synthesize/addSynthesizeAndProjects'),
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
                        this.dataForm.synthesizeName="",
                            this.dataForm.synthesizPrice="",
                            this.dataForm.detailed=''
                        /*跳转页面*/
                        this.$router.replace({ name: 'test_synthesize-synthesize' })
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