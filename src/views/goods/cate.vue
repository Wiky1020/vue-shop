<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addCate">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table class="tree" :data="getCateList" :columns="columns" border stripe :show-index='true' index-text='#' :show-row-hover='false' :selection-type='false' :expand-type='false'>
        <template v-slot:isok="scope">
          <i class="el-icon-success" style="color:lightgreen" v-if="scope.row.cat_deleted==false"></i>
          <i class="el-icon-error" style="color:red" v-else></i>
        </template>
        <template v-slot:order="scope">
          <el-tag v-if='scope.row.cat_level==0' type="primary">一级</el-tag>
          <el-tag v-else-if='scope.row.cat_level==1' type="success">二级</el-tag>
          <el-tag v-else type="warning">三级</el-tag>
        </template>
        <template slot="opt">
          <el-button type="primary" class="el-icon-edit" size="mini"> 编辑</el-button>
          <el-button type="danger" class="el-icon-delete" size="mini" >删除</el-button>
        </template>
      </tree-table>
<!-- 分页 -->
        <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryinfo.pagenum"
      :page-sizes="[3, 5,10, 15]"
      :page-size="queryinfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addDialogVisible" width="50%" @close="closeDialog">
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-ruleForm">
    <el-form-item label="分类名称" prop="cat_name">
      <el-input v-model="addForm.cat_name"></el-input>
    </el-form-item>
     <el-form-item label="父级分类" prop="cat_pid">
       <!-- 级联选择器 -->
      <el-cascader v-model="selectedKeys" :options="getCateList" :props="parentProps" @change="handleChange" clearable></el-cascader>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCateinfo">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      getCateList: [],
      queryinfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // 各列数据
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isok'
      }, {
        label: '排序',
        type: 'template',
        template: 'order'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }
      ],
      // 控制添加分类的显示与隐藏
      addDialogVisible: false,
      // 添加表单的数据
      addForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      // 表单验证规则
      addFormRules: {
        cat_name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }],
        cat_pid: [{ required: true, message: '请选择父级分类', trigger: 'blur' }]
      },
      // 父级分类的prop
      parentProps: {
        expandTrigger: 'hover',
        checkStrictly: 'true',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 存储选择的id
      selectedKeys: []
    }
  },
  created () {
    this.getCateLists()
  },
  methods: {
    // 获取商品分类
    async getCateLists () {
      const { data: res } = await this.$http.get('categories', { params: this.queryinfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.getCateList = res.data.result
      this.total = res.data.total
      console.log(this.getCateList)
    },
    // 分页
    handleSizeChange (newSize) {
      this.queryinfo.pagesize = newSize
      this.getCateLists()
    },
    handleCurrentChange (newNum) {
      this.queryinfo.pagenum = newNum
      this.getCateLists()
    },
    // 添加分类
    addCate () {
      this.addDialogVisible = true
    },
    handleChange () {
      console.log(this.selectedKeys)
      this.addForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
      this.addForm.cat_level = this.selectedKeys.length
      console.log(this.addForm)
    },
    // 监听表单的重置
    closeDialog () {
      this.$refs.addFormRef.resetFields()
      this.selectedKeys = []
    },
    // 添加分类对表单预校验
    addCateinfo () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('添加分类失败')
        this.$message.success('添加分类成功')
        this.getCateLists()
        this.addDialogVisible = false
      })
    }
  }
}
</script>
<style lang='less' scoped>
.tree{
  margin: 15px 0px;
}
</style>
