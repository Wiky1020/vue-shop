<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert class="alert" title="注意：只允许选择第三级分类设置相关参数" type="warning" show-icon></el-alert>
      <el-row>
        <el-col>
          <span>选择商品分类：</span>
           <el-cascader clearable v-model="selectedKeys" :options="getCateList" :props="defaultProp" @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!-- 参数数据 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" :disabled="isdisabled" size="mini" @click="addInfo">添加参数</el-button>
          <!-- 参数数据 -->
          <el-table :data="manyList" border stripe class="table" >
          <el-table-column  type="expand" >
            <!-- 渲染参数数据 -->
            <template v-slot="scope">
              <el-tag v-for='(item,i) in scope.row.attr_vals' :key="i" closable @close="removeTag(scope.row,i)">{{item}}</el-tag>
              <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
              </el-input>
              <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
            </template>
          </el-table-column>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column  label="参数名称" prop="attr_name"></el-table-column>
          <el-table-column  label="操作">
            <template v-slot="scope">
              <el-button type="primary" size="mini" class="el-icon-edit" @click="editInfo(scope.row)">编辑</el-button>
              <el-button type="danger" size="mini" class="el-icon-delete" @click="deleteInfo(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isdisabled" size="mini" @click="addInfo">添加属性</el-button>
          <!-- 参数数据 -->
           <el-table :data="onlyList" border stripe class="table" >
          <el-table-column  type="expand" >
             <!-- 渲染属性数据 -->
            <template v-slot="scope">
              <el-tag v-for='(item,i) in scope.row.attr_vals' :key="i" closable @close="removeTag(scope.row,i)">{{item}}</el-tag>
              <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
              </el-input>
              <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
            </template>
          </el-table-column>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column  label="参数名称" prop="attr_name"></el-table-column>
          <el-table-column  label="操作">
           <template v-slot="scope">
              <el-button type="primary" size="mini" class="el-icon-edit" @click="editInfo(scope.row)">编辑</el-button>
              <el-button type="danger" size="mini" class="el-icon-delete" @click="deleteInfo(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数或属性的对话框 -->
    <el-dialog
  :title="'添加'+titleText()"
  :visible.sync="addDialogVisible"
  width="50%" @close="resetAdd">
   <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-ruleForm">
   <el-form-item :label="titleText()" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
   </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog>

 <!-- 修改参数或属性的对话框 -->
    <el-dialog
  :title="'修改'+titleText()"
  :visible.sync="editDialogVisible"
  width="50%" @close="resetEdit">
   <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
   <el-form-item :label="titleText()" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
   </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 获取商品分类
      getCateList: [],
      // 定义属性
      defaultProp: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      selectedKeys: [],
      activeName: 'many',
      isdisabled: true,
      // 分类id
      cateId: 0,
      // 动态参数
      manyList: [],
      // 静态属性
      onlyList: [],
      // 控制添加参数和属性显示与隐藏
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      editForm: {
      },
      addFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      editFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      }

    }
  },
  created () {
    this.getCateLists()
  },
  methods: {
    // 获取商品列表
    async getCateLists () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类列表失败')
      this.getCateList = res.data
      // console.log(this.getCateList)
    },
    // 父级id
    handleChange () {
      this.getParamsList()
    },
    // tabs发生变化触发函数
    handleClick () {
      this.getParamsList()
    },
    // 根据id获取分类参数
    async getParamsList () {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyList = []
        this.onlyList = []
        this.isdisabled = true
        return
      }
      this.isdisabled = false
      this.cateId = this.selectedKeys[2]
      const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取分类失败')
      this.$message.success('获取分类成功')
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyList = res.data
        console.log(this.cateId)
      } else if (this.activeName === 'only') {
        this.onlyList = res.data
      }
    },
    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 回车失去焦点会触发
    handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveTag(row)
    },
    async saveTag (row) {
      // 存储数据库
      const { data: res } = await this.$http.put('categories/' + this.cateId + '/attributes/' + row.attr_id, { attr_name: row.attr_name, attr_sel: this.activeName, attr_vals: row.attr_vals.join(',') })
      if (res.meta.status !== 200) return this.$message.error('修改失败')
      this.$message.success('修改成功')
    },
    // 删除标签
    removeTag (row, i) {
      row.attr_vals.splice(i, 1)
      this.saveTag(row)
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    },
    // 添加参数或属性
    addInfo () {
      this.addDialogVisible = true
    },

    editInfo (row) {
      this.editForm = row
      this.editDialogVisible = true
    },
    // 添加数据
    addParams () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories/' + this.cateId + '/attributes', { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
        console.log(res)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.getParamsList()
        this.addDialogVisible = false
      })
    },
    // 重置表单
    resetAdd () {
      this.$refs.addFormRef.resetFields()
    },
    // 修改
    resetEdit () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改数据
    editParams () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.cateId + '/attributes/' + this.editForm.attr_id, { attr_name: this.editForm.attr_name, attr_sel: this.activeName })
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('编辑失败')
        this.$message.success('修改成功')
        this.getParamsList()
        this.editDialogVisible = false
      }
      )
    },
    // 删除
    async deleteInfo (row) {
      const result = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') return this.$message.info('已取消了删除')
      const { data: res } = await this.$http.delete('categories/' + this.cateId + '/attributes/' + row.attr_id)
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getParamsList()
    }
  }
}
</script>
<style lang="less" scoped>
.alert{
  margin-bottom: 15px;
}
.table{
  margin-top: 15px;
  font-size: 12px;
}
.el-tag{
  margin: 0 8px;
}
.input-new-tag{
  width: 120px;
}
</style>
