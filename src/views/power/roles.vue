<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/homr' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary"  @click="addRoleInfo">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="getRoleList">
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row :class="['btnbottom',i==0?'btntop':'']" v-for="(item,i) in scope.row.children" :key="item.id">
              <el-col :span="5" class="colflex">
                <el-tag closable type="primary" @close="removeTag(scope.row,item.id)">{{ item.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row :class="[i1==0?'':'btntop']" v-for="(item1,i1) in item.children" :key="item1.id">
                  <el-col :span="6">
                    <el-tag  closable type="success" @close="removeTag(scope.row,item1.id)">{{ item1.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                      <el-tag closable class="tag" type="warning" v-for="(item2) in item1.children" :key="item2.id" @close="removeTag(scope.row,item2.id)">{{ item2.authName }}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
             <el-button type="primary" icon="el-icon-edit" size="mini" @click="editRoleInfo(scope.row)"
              >编辑</el-button
            >
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteRole(scope.row.id)"
              >删除</el-button
            >
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="addRights(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的权限对话框 -->
    <el-dialog
  title="分配权限"
  :visible.sync="addRightsDialogVisible"
  width="50%" @close='resetArr'>
 <el-tree :data="getRightsLists" ref="treeRef" :props="getRightsProps" show-checkbox default-expand-all node-key="id" :default-checked-keys="defaultArray"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addRightsDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addRightsInfo">确 定</el-button>
  </span>
</el-dialog>
<!-- 添加角色的对话框 -->
    <el-dialog
  title="添加角色"
  :visible.sync="addDialogVisible"
  width="50%">
  <el-form :model="addFormInfo" :rules="addRormRules" ref="addFormRef" label-width="100px">
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="addFormInfo.roleName"></el-input>
  </el-form-item>
   <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="addFormInfo.roleDesc"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="resetAddFrom">取 消</el-button>
    <el-button type="primary" @click="addRole">确 定</el-button>
  </span>
</el-dialog>

  <!-- 修改角色的对话框 -->
    <el-dialog
  title="修改角色"
  :visible.sync="editDialogVisible"
  width="50%">
  <el-form :model="editFormInfo" :rules="editRormRules" ref="editFormRef" label-width="100px">
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="editFormInfo.roleName"></el-input>
  </el-form-item>
   <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="editFormInfo.roleDesc"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="resetEditFrom">取 消</el-button>
    <el-button type="primary" @click="editRole">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 角色数据列表
      getRoleList: [],
      // 权限数据列表
      getRightsLists: [],
      // 控制对话框显示与隐藏
      addRightsDialogVisible: false,
      // 权限数据的展示
      getRightsProps: {
        label: 'authName',
        children: 'children'
      },
      // 选中节点的数组
      defaultArray: [],
      // 角色id
      roleId: 0,
      // 添加角色的对话框的显示与隐藏
      addDialogVisible: false,
      // 添加角色的信息
      addFormInfo: {
        roleName: '',
        roleDesc: ''
      },
      // 角色规则
      addRormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }],
        roleDesc: [{ required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 2 到 15 个字符', trigger: 'blur' }]
      },
      // 修改表单
      editDialogVisible: false,
      // 修改表单数据
      editFormInfo: {},
      // 修改表单数据规则
      editRormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }],
        roleDesc: [{ required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 2 到 15 个字符', trigger: 'blur' }]
      }
    }
  },
  created () {
    this.getRoles()
  },
  methods: {
    async getRoles () {
      const { data: res } = await this.$http.get('/roles')
      if (res.meta.status !== 200) { return this.$message.error('获取角色列表失败') }
      this.getRoleList = res.data
    },
    // 点击标签移除标签
    async removeTag (role, id) {
      const result = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') return this.$message.info('已取消了删除操作')
      console.log(role, id)
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${id}`)
      if (res.meta.status !== 200) return this.$message.error('修改用户权限失败')
      this.$message.success('删除用户权限成功')
      role.children = res.data
    },
    // 获取权限列表
    async getRightsList () {
      const { data: res } = await this.$http.get('/rights/tree')
      if (res.meta.status !== 200) { return this.$message.error('获取权限列表失败') }
      this.getRightsLists = res.data
    },
    // 点击分配权限
    addRights (role) {
      this.roleId = role.id
      this.getRightsList()
      this.getRigthsId(role, this.defaultArray)
      this.addRightsDialogVisible = true
    },
    // 点击确定按钮
    addRightsInfo () {
      this.getRightsIdArray()

      this.addRightsDialogVisible = false
    },
    // 定义一个获取所有权限的函数
    getRigthsId (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      } else {
        node.children.forEach(item => {
          return this.getRigthsId(item, arr)
        })
      }
    },
    // 重置分配权限
    resetArr () {
      this.defaultArray = []
    },
    // 获取所有节点的数组
    async getRightsIdArray () {
      const array = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const str = array.join(',')
      const { data: res } = await this.$http.post('roles/' + this.roleId + '/rights', { rids: str })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('角色授权失败')
      this.$message.success('角色授权成功')
      this.getRoles()
    },
    // 添加角色
    addRoleInfo () {
      this.addDialogVisible = true
    },
    // 添加角色的表单重置
    resetAddFrom () {
      this.$refs.addFormRef.resetFields()
      this.addDialogVisible = false
    },
    // 添加角色对表单预校验
    addRole () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('/roles', this.addFormInfo)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败')
        this.$message.success('添加成功')

        this.getRolesList()
        this.addDialogVisible = false
      })
    },
    // 编辑角色
    editRoleInfo (role) {
      this.editFormInfo = role
      this.editDialogVisible = true
    },
    // 修改角色的表单重置
    resetEditFrom () {
      this.$refs.editFormRef.resetFields()
      this.editDialogVisible = false
    },
    /// / 修改角色对表单预校验
    editRole () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put('/roles/' + this.editFormInfo.id, this.editFormInfo)
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('修改角色失败')
        this.$message.success('修改角色成功')
        this.getRolesList()
        this.editDialogVisible = false
      })
    },
    // 删除角色
    async deleteRole (id) {
      console.log(id)
      const result = await this.$confirm('此操作将永久删除该用户角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') return this.$message.info('取消了删除角色')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除角色失败')
      this.$message.success('删除成功')
      this.getRolesList()
    }
  }

}
</script>
<style lang="less" scoped>
.el-tag{
  margin: 8px 0;
}
.btntop{
  border-top: 1px solid rgb(213, 207, 207);
}
.btnbottom{
  border-bottom: 1px solid rgb(213, 207, 207);
}
.tag{
  margin: 8px 5px;
}
.el-row{
  display: flex;
  align-items: center;
}
</style>
