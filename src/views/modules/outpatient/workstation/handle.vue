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
export default {
  props:['patient'],
  name:'Inspection',
  data(){
    return{
      onemodel:{},
      totalprice:0.000,
      ref:[],
      checkmodels:[],
      check:{},
      demandVisible:false,
      dialogTableVisible:false,
      activeName:'first',
      isclose:true,
      checkList:[],
      mainwidth:"65%",
      activeNames: ['1'],
      total:0,
      search:'',
      listQuery: {
        code:null,
        name:null,
        format:null,
        price:null,
        expClassId: null,
        deptId: null,
        mnemonicCode:null,
        recordType: 3,
        createDate: null,
        status:1,
        pageSize:1000,
        pageNum:1
      },
      record:[]
    };
  },
   watch:{
    'patient' : function(newVal){
      this.patient = newVal
    },
  },
  methods:{
    addfreitem(val){
      this.selectCheck(val)
    },
    addModel(val){
      this.$confirm('是否确定将 模板:'+val.name+' 中的内容导入该患者的检查中', '导入模板', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
          val.nondruglist.forEach(item=>{
            item.status = -1
            let flag = 1
            this.record.forEach(com=>{
              if(com.id===item.id){
                flag = 0
              }
            })
            if(flag)
              this.record.push(item)
          })
        })
    },
    selectModel(val){
      this.onemodel = val
      console.log(this.onemodel)
    },
    refresh(){
      this.$confirm('未开立的项目都将清除,确认刷新?', '刷新', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        })
    },
    handleSelectionChange(val){
      this.ref = val
      this.totalprice = 0.00
      this.ref.forEach(item=>{
        this.totalprice+=item.price
      })
      this.totalprice = this.totalprice.toFixed(2)
    },
    submitDemand(){
      this.record.forEach(item=>{
        if(item.id===this.check.id){
          item.aim = this.check.aim
          item.demand = this.check.demand
          item.clinicalImpression = this.check.clinicalImpression
          item.clinicalDiagnosis = this.check.clinicalDiagnosis
          item.checkParts = this.check.checkParts
        }
      })
      this.demandVisible = false
    },
    selectCheck(val){
      val.status=0
      this.$confirm('是否添加 '+val.name+' 到该患者?', '添加检查', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'success'
        }).then(()=>{
          let flag = 1
          val.status=-1
          this.record.forEach(item=>{
            if(item.id===val.id){
              flag=0
            }
          })
          if(flag)
            this.record.push(val)
          else
            alert('已存在该检查项目！')
          this.dialogTableVisible = false
        })
    },
    addcheck(){
      this.dialogTableVisible = true
    },
    delcheck(){
      this.record = this.record.filter(item=>{
        if(item.status===0)
          if(this.ref.includes(item))
            return false
        return true
      })

    },
    controlfast(){
      this.isclose=!this.isclose
      if(this.mainwidth==="65%")
        this.mainwidth="80%"
      else
        this.mainwidth="65%"
    }
  }
}
</script>

