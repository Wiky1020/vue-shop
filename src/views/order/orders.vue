<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/homr' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable v-model="ordrInfo.query" @clear="getOrdersList">
          <el-button slot="append" icon="el-icon-search" @click="getOrdersList"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="getOrderList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price" width="120px"></el-table-column>
        <el-table-column label="是否付款" prop="order_pay" width="120px">
          <template v-slot="scope">
            <el-tag type="danger" v-if="scope.row.order_pay==='0'">未付款</el-tag>
            <el-tag type="success" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" width="120px"></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="200px">
          <template v-slot="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150px">
          <template>
            <el-button type="primary" size="mini" class="el-icon-edit" @click="editInfo"></el-button>
            <el-button type="success" size="mini" class="el-icon-location" @click="getInfo"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
       <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="ordrInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="ordrInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 修改地址的对话框 -->
    <el-dialog
  title="修改地址"
  :visible.sync="addressDialogVisible"
  width="50%">
  <el-form :model="adressForm" :rules="adressFormRule" ref="adressFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item label="省市区/县" prop="address1">
   <el-cascader
    v-model="adressForm.address1"
    :options="cityData"
    :props="addressProp"
    @change="handleChange"></el-cascader>
  </el-form-item>
    <el-form-item label="详细地址" prop="address2">
    <el-input v-model="adressForm.address2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="resetInfo">取 消</el-button>
    <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
  </span>
</el-dialog>
<!-- 获取物流信息的对话框 -->
  <el-dialog
  title="修改地址"
  :visible.sync="KudiDialogVisible"
  width="50%">
  <el-timeline >
    <el-timeline-item
      v-for="(activity, index) in dataInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
  </el-dialog>
  </div>
</template>
<script>
import cityData from './citydata'
export default {
  data () {
    return {
      getOrderList: [],
      ordrInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      addressDialogVisible: false,
      KudiDialogVisible: false,
      cityData,
      selectkey: [],
      addressProp: {
        expandTrigger: 'hover',
        label: 'label',
        value: 'value',
        children: 'children'
      },
      adressForm: {
        address1: [],
        adress2: ''
      },
      adressFormRule: {
        address1: [{ required: true, message: '请选择省市区县', trigger: 'blur' }],
        address2: [{ required: true, message: '请输入详细地址', trigger: 'blur' }]
      },
      dataInfo: [{
        time: '2018-05-10 09:39:00',
        ftime: '2018-05-10 09:39:00',
        context: '已签收,感谢使用顺丰,期待再次为您服务',
        location: ''
      },
      {
        time: '2018-05-10 08:23:00',
        ftime: '2018-05-10 08:23:00',
        context: '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
        location: ''
      },
      {
        time: '2018-05-10 07:32:00',
        ftime: '2018-05-10 07:32:00',
        context: '快件到达 [北京海淀育新小区营业点]',
        location: ''
      },
      {
        time: '2018-05-10 02:03:00',
        ftime: '2018-05-10 02:03:00',
        context: '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
        location: ''
      },
      {
        time: '2018-05-09 23:05:00',
        ftime: '2018-05-09 23:05:00',
        context: '快件到达 [北京顺义集散中心]',
        location: ''
      },
      {
        time: '2018-05-09 21:21:00',
        ftime: '2018-05-09 21:21:00',
        context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
        location: ''
      },
      {
        time: '2018-05-09 13:07:00',
        ftime: '2018-05-09 13:07:00',
        context: '顺丰速运 已收取快件',
        location: ''
      },
      {
        time: '2018-05-09 12:25:03',
        ftime: '2018-05-09 12:25:03',
        context: '卖家发货',
        location: ''
      },
      {
        time: '2018-05-09 12:22:24',
        ftime: '2018-05-09 12:22:24',
        context: '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
        location: ''
      },
      {
        time: '2018-05-08 21:36:04',
        ftime: '2018-05-08 21:36:04',
        context: '商品已经下单',
        location: ''
      }]
    }
  },
  created () {
    this.getOrdersList()
  },
  methods: {
    async getOrdersList () {
      const { data: res } = await this.$http.get('orders', { params: this.ordrInfo })
      if (res.meta.status !== 200) return this.$message.error('获取订单列表失败')
      this.$message.success('获取订单列表成功')
      this.getOrderList = res.data.goods
      this.total = res.data.total
      console.log(this.getOrderList)
    },
    // 分页
    handleSizeChange (newSize) {
      this.ordrInfo.pagesize = newSize
      this.getOrdersList()
    },
    handleCurrentChange (newNum) {
      this.ordrInfo.pagenum = newNum
      this.getOrdersList()
    },
    // 修改地址信息
    editInfo () {
      console.log(this.cityData)
      this.addressDialogVisible = true
    },
    handleChange () {},
    // 重置表单
    resetInfo () {
      this.$refs.adressFormRef.resetFields()
      this.addressDialogVisible = false
    },
    // 获取物流信息
    getInfo () {
      this.KudiDialogVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-table{
  margin-top: 15px;
}
.el-pagination{
  margin-top: 15px ;
}
</style>
