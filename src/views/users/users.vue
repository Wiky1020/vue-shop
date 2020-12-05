<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/homr' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
          <el-col :span="8">
            <el-input placeholder="请输入内容" v-model="userinfo.query" clearable @clear='getUsersList'>
                 <el-button slot="append" icon="el-icon-search" @click="getUsersList"></el-button>
            </el-input>
          </el-col>
          <el-col :span="4"><el-button type="primary" @click="addUsers">添加用户</el-button></el-col>
      </el-row>
      <el-table stripe  :data='usersList'>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template v-slot='scope'>
            <el-switch v-model="scope.row.mg_state" @change="chageStatus(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
           <template v-slot='scope'>
             <!-- 编辑按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="editUser(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUser(scope.row.id)"></el-button>
             <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable='false'>
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
             </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
       <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="userinfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="userinfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加用户提示框 -->
    <el-dialog
  title="添加用户"
  :visible.sync="addVisible"
  width="50%" @close="resetUserInfo">
  <el-form :model="addRuleForm" :rules="addUserrules" ref="addFormRef" label-width="100px">
  <el-form-item label="用户名" prop="username">
    <el-input v-model="addRuleForm.username"></el-input>
  </el-form-item>
  <el-form-item label="密码" prop="password">
    <el-input v-model="addRuleForm.password" type="password"></el-input>
  </el-form-item>
  <el-form-item label="邮箱" prop="email">
    <el-input v-model="addRuleForm.email"></el-input>
  </el-form-item>
  <el-form-item label="手机" prop="mobile">
    <el-input v-model="addRuleForm.mobile"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="resetUserInfo">取 消</el-button>
    <el-button type="primary" @click="addUserInfo">确 定</el-button>
  </span>
</el-dialog>
<!-- 编辑用户提示框 -->
    <el-dialog
  title="修改用户信息"
  :visible.sync="editVisible"
  width="50%">
  <el-form :model="editRuleForm" :rules="editUserrules" ref="editFormRef" label-width="100px">
  <el-form-item label="用户名" >
    <el-input v-model="editRuleForm.username" :disabled="true"></el-input>
  </el-form-item>
  <el-form-item label="邮箱" prop="email">
    <el-input v-model="editRuleForm.email"></el-input>
  </el-form-item>
  <el-form-item label="手机" prop="mobile">
    <el-input v-model="editRuleForm.mobile"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="removeEdit">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
</el-dialog>
</div>
</template>
<script>
export default {
  data () {
    // 自定义邮箱的验证规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) return callback()
      callback(new Error('请输入合法的邮箱'))
    }
    // 定义手机的验证规则
    var MobileEmail = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) return callback()
      callback(new Error('请输入合法的手机'))
    }
    return {
      userinfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      usersList: [],
      total: 0,
      addVisible: false,
      editVisible: false,
      addRuleForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editRuleForm: {
      },
      addUserrules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' },
          { validator: MobileEmail, trigger: 'blur' }
        ]
      },
      editUserrules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' },
          { validator: MobileEmail, trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getUsersList()
  },
  methods: {
    async  getUsersList () {
      const { data: res } = await this.$http.get('/users', { params: this.userinfo })
      if (res.meta.status !== 200) return this.$message.error('获取列表失败')
      this.usersList = res.data.users
      this.total = res.data.total
    },
    async chageStatus (status) {
      const { data: res } = await this.$http.put(`users/${status.id}/state/${status.mg_state}`)
      if (res.meta.status !== 200) return this.$message.error('修改用户状态失败')
      this.$message.success('修改用户状态成功')
      this.getUsersList()
    },
    handleSizeChange (newSize) {
      console.log(newSize)
      this.userinfo.pagesize = newSize
      this.getUsersList()
    },
    handleCurrentChange (newPage) {
      console.log(newPage)
      this.userinfo.pagenum = newPage
      this.getUsersList()
    },
    // 添加用户
    addUsers () {
      this.addVisible = true
    },
    // 重置表单
    resetUserInfo () {
      this.addVisible = false
      this.$refs.addFormRef.resetFields()
    },
    // 添加用户进行预校验
    addUserInfo () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addRuleForm)
        console.log(res)
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        this.$message.success('添加用户成功')
        this.addVisible = false
        this.getUsersList()
      })
    },
    // 编辑用户
    async editUser (id) {
      const { data: res } = await this.$http.get('/users/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取用户失败')
      this.editRuleForm = res.data
      this.editVisible = true
    },
    // 编辑表单重置
    removeEdit () {
      this.editVisible = false
      this.$refs.editFormRef.resetFields()
    },
    // 提交表单数据对表单预校验
    editUserInfo () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put('/users/' + this.editRuleForm.id, { email: this.editRuleForm.email, mobile: this.editRuleForm.mobile })
        if (res.meta.status !== 200) return this.$message.error('更新用户失败')
        this.$message.success('更新成功')
        this.editVisible = false
        this.getUsersList()
      })
    },
    // 删除用户
    async deleteUser (id) {
      console.log(id)
      const result = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') return this.$message.info('取消了删除操作')
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      this.$message.success('删除用户成功')
      this.getUsersList()
    }

  }
}
</script>
<style lang="less" scoped>

</style>
