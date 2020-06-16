<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable>
            <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderList" style="width: 100%" border stripe>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column prop="order_number" label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格" width="180">
        </el-table-column>
        <el-table-column label="是否付款" width="180">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货" width="180">
        </el-table-column>
        <el-table-column label="下单时间" width="180">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="140">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditAddressDialog"></el-button>
            <el-button type="success" icon="el-icon-location-outline" size="mini" @click="showProgressDialog">
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[10, 15, 20]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
    <!-- 修改地址对话框 -->
    <el-dialog title="修改地址" :visible.sync="isEditAddressDialogVisible" width="40%" @closed="editAddressDialogClosed">
      <el-form :model="editAddressForm" :rules="editAddressFormRules" ref="editAddressFormRef" label-width="90px">

        <el-form-item label="省市区/县" prop="location1">
          <el-cascader v-model="editAddressForm.location1" :options="cityData" @change="handleCascaderChange"
            :props="cascaderProps"></el-cascader>
        </el-form-item>

        <el-form-item label="详细地址" prop="location2">
          <el-input v-model="editAddressForm.location2"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isEditAddressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="isEditAddressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流进度对话框 -->
    <el-dialog title="物流进度" :visible.sync="isProgressDialogVisible" width="40%">
      <el-timeline >
        <el-timeline-item v-for="(activity, index) in progressDataList" :key="index" :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from '../../plugins/citydata'
export default {
  data: function () {
    return {
      orderList: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      isEditAddressDialogVisible: false,
      editAddressForm: {
        location2: '',
        location1: []
      },
      editAddressFormRules: {
        location1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        location2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      cityData: cityData,
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'value',
        label: 'label',
        children: 'children'
      },
      isProgressDialogVisible: false,
      progressDataList: []
    }
  },
  created: function () {
    this.getOrderList()
  },
  methods: {
    getOrderList: async function () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange: function (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange: function (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showEditAddressDialog: function () {
      this.isEditAddressDialogVisible = true
    },
    handleCascaderChange: function () { },
    editAddressDialogClosed: function () {
      this.$refs.editAddressFormRef.resetFields()
    },
    showProgressDialog: function () {
      this.getProgressDataList()
      this.isProgressDialogVisible = true
    },
    getProgressDataList: async function () {
      const id = '1106975712662'
      const { data: res } = await this.$http.get('/kuaidi/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.progressDataList = res.data
      console.log(this.progressDataList)
    }
  }
}
</script>

<style lang="less" scoped>

</style>
