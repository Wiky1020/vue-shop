<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/homr' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-table :data="getRightsList">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="level">
          <template v-slot="scope">
            <el-button type="primary" plain v-if="scope.row.level=='0'">一级</el-button>
            <el-button type="success" plain v-else-if="scope.row.level=='1'">二级</el-button>
            <el-button type="warning" plain v-else-if="scope.row.level=='2'">三级</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      getRightsList: []
    }
  },
  created () {
    this.getRights()
  },
  methods: {
    async getRights () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.getRightsList = res.data
    }
  }
}
</script>
<style lang="less" scoped></style>
