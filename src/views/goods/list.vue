<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable v-model="queryInfo.query" @clear='getGoodsLists'>
          <el-button slot="append" icon="el-icon-search" @click="getGoodsLists"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addInfo">添加商品</el-button>
        </el-col>
      </el-row>
       <el-table :data="getGoodsList" style="width: 100%" border stripe>
      <el-table-column type="index" label="#"></el-table-column>
      <el-table-column  label="商品名称" prop="goods_name"></el-table-column>
      <el-table-column  label="商品价格" prop="goods_price" width="95px"></el-table-column>
      <el-table-column  label="商品重量" prop="goods_weight" width="70px"></el-table-column>
      <el-table-column  label="创建时间" prop="add_time" width="180px">
        <template v-slot="scope">{{ scope.row.add_time | dateFormat}}</template>
      </el-table-column>
      <el-table-column  label="操作" width="140px">
        <template>
          <el-button class="el-icon-edit" type="primary" size="mini"></el-button>
          <el-button class="el-icon-delete" type="danger" size="mini"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="this.queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="this.queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      getGoodsList: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0
    }
  },
  created () {
    this.getGoodsLists()
  },
  methods: {
    async getGoodsLists () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.$message.success('获取成功')
      this.getGoodsList = res.data.goods
      this.total = res.data.total
      console.log(res)
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsLists()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsLists()
    },
    // 点击添加商品
    addInfo () {
      this.$router.push('/goods/add')
    }

  }
}
</script>
<style lang="less" scoped>
.el-table{
  margin-top: 15px;
}
</style>
