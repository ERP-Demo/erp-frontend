<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.userName" placeholder="用户名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:user:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('sys:user:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        prop="userId"
        header-align="center"
        align="center"
        width="80"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="用户名">
      </el-table-column>
      <el-table-column
        prop="email"
        header-align="center"
        align="center"
        label="邮箱">
      </el-table-column>
      <el-table-column
        prop="mobile"
        header-align="center"
        align="center"
        label="手机号">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">禁用</el-tag>
          <el-tag v-else size="small">正常</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        width="180"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="170"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="dialogVisible = true,addHandle(scope.row.userId)">新增</el-button>
          <el-button v-if="isAuth('sys:user:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.userId)">修改</el-button>
          <el-button v-if="isAuth('sys:user:delete')" type="text" size="small" @click="deleteHandle(scope.row.userId)">删除</el-button>
          <el-button type="text" size="small" @click="dialogVisible1 = true,toviwe(scope.row.userId)">查看</el-button>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

    <el-dialog
            title="提示"
            :visible.sync="dialogVisible"
            width="66%"
            :before-close="handleClose">
      <el-table
              ref="multipleTable"
              :data="tableData"
              tooltip-effect="dark"
              style="width: 100%"
              @selection-change="handleSelectionChange">
        <el-table-column
                type="selection"
                width="55">
        </el-table-column>
        <el-table-column
                prop="departmentId"
                header-align="center"
                align="center"
                label="部门编号">
        </el-table-column>
        <el-table-column
                prop="departmentName"
                header-align="center"
                align="center"
                label="部门名称">
        </el-table-column>
      </el-table>
      <div style="margin-top: 20px">
        <el-button @click="toggleSelection()">取消选择</el-button>
      </div>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false,confirmf()">确 定</el-button>
  </span>
    </el-dialog>
    <el-dialog
            title="提示"
            :visible.sync="dialogVisible1"
            width="66%"
            :before-close="handleClose">
      <el-table
              ref="multipleTable"
              :data="departmentData"
              tooltip-effect="dark"
              style="width: 100%"
              @selection-change="handleSelectionChange">
        <el-table-column
                type="selection"
                width="55">
        </el-table-column>
        <el-table-column
                prop="departmentId"
                header-align="center"
                align="center"
                label="部门编号">
        </el-table-column>
        <el-table-column
                prop="departmentName"
                header-align="center"
                align="center"
                label="部门名称">
        </el-table-column>
        <el-table-column
                fixed="right"
                header-align="center"
                align="center"
                width="160"
                label="操作">
          <template slot-scope="scope">
            <el-button type="text" size="small" @click="delHandle(scope.row.departmentId)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible1 = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible1 = false">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
import AddOrUpdate from './user-add-or-update'
export default {
  data () {
    return {
      dataForm: {
        userName: ''
      },
      departmentData:[],
      dataList: [],
      tableData:[],
      dialogVisible: false,
      dialogVisible1: false,
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      uid:0,
      departmentId:0,
      userId:0
    }
  },
  components: {
    AddOrUpdate
  },
  activated () {
    this.getDataList(),
    this.getData()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/ucenter/user/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'username': this.dataForm.userName
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
    addHandle(id){
      this.uid=id;
    },
    //多表数据
    toviwe(id){
      this.userId=id
      this.$http({
        url: this.$http.adornUrl('/users_department/Department/all/'+id),
        method: 'get',
        data: this.$http.adornData()
      }).then(({data}) => {
        if (data && data.code === 200) {
            this.departmentData = data.list
          console.log("数据"+data.list)
        } else {
          this.departmentData = []
        }
      })
    },
    //删除多表
    delHandle(id){
      this.$confirm(`确定要删除！`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/users_department/Department/deletebyid/'+id+"/"+this.userId),
          method: 'get',
          data: this.$http.adornData()
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.toviwe(this.userId)
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
    //取消全选
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    //提交
    confirmf(){
      var ids=this.multipleSelection.map(item =>{return item.departmentId}).join(",")
      this.$http({
        url: this.$http.adornUrl('/users_department/Department/add/'+this.uid+"/"+ids),
        method: 'get',
      }).then(({data}) => {
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
    },
    //获取复选框
    handleSelectionChange(val) {
      this.multipleSelection = val;
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
    // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    //获取部门
    getData () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/department/Department/list'),
        method: 'get',
        params: this.$http.adornParams({})
      }).then(({data}) => {
        if (data && data.code === 200) {
          this.tableData = data.page.list
        } else {
          this.tableData = []
        }
        this.dataListLoading = false
      })
    },
    // 删除
    deleteHandle (id) {
      var userIds = id ? [id] : this.dataListSelections.map(item => {
        return item.userId
      })
      this.$confirm(`确定对[id=${userIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/ucenter/user/delete'),
          method: 'post',
          data: this.$http.adornData(userIds, false)
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {})
    }
  }
}
</script>
