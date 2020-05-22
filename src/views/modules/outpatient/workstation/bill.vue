<template>
  <!-- 患者账单 -->
  <div>
    <el-table :data="arrayList" height="530px" @selection-change="handleSelectionChange">
      <el-table-column label="项目名" prop="name"></el-table-column>
      <el-table-column label="数量" prop="num"></el-table-column>
      <el-table-column label="单价" prop="price"></el-table-column>
      <el-table-column label="总金额" prop="totalprice">
        <template slot-scope="scope">{{ scope.row.num * scope.row.price }}</template>
      </el-table-column>
      <el-table-column label="类型" prop="type">
        <template slot-scope="scope">
          <span v-if="scope.row.type===0">检查</span>
          <span v-if="scope.row.type===1">成药处方</span>
          <span v-if="scope.row.type===2">处置</span>
        </template>
      </el-table-column>
      <el-table-column label="状态">
        <template slot-scope="scope">
          <el-tag type="danger" v-if="scope.row.status===0">作废</el-tag>
          <el-tag type="warning" v-if="scope.row.status===1">未缴费</el-tag>
          <el-tag type="success" v-if="scope.row.status===2">已缴费</el-tag>
          <el-tag type="success" v-if="scope.row.status===3">已登记</el-tag>
          <el-tag type="success" v-if="scope.row.status===4">已执行</el-tag>
          <el-tag type="danger" v-if="scope.row.status===5">已退费</el-tag>
          <el-tag type="danger" v-if="scope.row.status===6">已过期</el-tag>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
export default {
  name: 'Bill',
  props: ['patient', 'registerId'],
  watch:{
    'patient' : function(newVal){
      this.patient = newVal
      this.getDataList1()
    },
  },
  data(){
    return{
      bill:[],
      arrayList: [],
      handleName: '',
      handlePrice: '',
      totalprice: ''
    };
  },
  methods: {

    handleSelectionChange(val) {
      this.ref = val
      this.totalprice = 0.00
      this.ref.forEach(item => {
        this.totalprice += item.price
      })
      this.totalprice = this.totalprice.toFixed(2)
    },

    //检查申请
    getDataList1 () {
      this.dataListLoading = true

      //检查申请
      this.$http({
        url: this.$http.adornUrl('/requirements/requirements/AllList/'+this.registerId),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 200) {
          data.list.map(item => {
            this.arrayList.push({
              'name': item.testSynthesizeName,
              'price': item.testSynthesizePrice,
              'status': item.status,
              'type': 0,
              'num': 1
            })
          })
        } else {
          this.dataList = []
        }
      }),

      //成药处方
      this.$http({
        url: this.$http.adornUrl('/prescription/prescription/queryByrId/'+this.registerId),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 200) {
          data.list2.map(item => {
            this.arrayList.push({
              'name': item.drugsName,
              'price': item.drugsPrice,
              'status': item.preStatus,
              'type': 1,
              'num': item.drugsNum
            })
          })
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })

      //处置申请
      this.$http({
        url: this.$http.adornUrl(`/patient_handle/handle/apply/list/${this.registerId}`),
        method: 'get'
      }).then(({data}) => {
        if (data && data.code === 200) {
          data.list.map(item => {
            this.handleName=item.patientHandle.handleName
            this.handlePrice=item.patientHandle.handlePrice
            this.arrayList.push({
              'name': this.handleName,
              'price': this.handlePrice,
              'status': item.status,
              'type': 2,
              'num': 1
            })
          })
        }
      })
    }
  },
}
</script>
