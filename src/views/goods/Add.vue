<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="添加商品信息" type="info" center show-icon></el-alert>
      <el-steps
        :space="200"
        :active="Number(activeIndex)"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-position="top"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-tabs tab-position="left" v-model="activeIndex" :before-leave="beforeTabs" @tab-click="tabClick">
          <el-tab-pane label="基本信息" name="0" >
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_params">
              <el-cascader v-model="addForm.goods_cat" :options="getcateList" :props="defaultProps" @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="(item,i) in manyList" :key="i">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="(item,i) in  onlyList" :key="i">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload class="upload-demo" :headers="headers" :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
             <quill-editor v-model="addForm.goods_introduce"></quill-editor>
             <el-button class="addbtn" type="primary" @click="addInfo">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 预览图片的对话框 -->
    <el-dialog
  title="预览效果"
  :visible.sync="prevewDialogVisible"
  width="50%">
  <img :src="url" alt="" style="width:100%;">
</el-dialog>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data () {
    return {
      // 商品分类数据
      getcateList: [],
      selectedKeys: [],
      defaultProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ]
      },
      // 动态参数数据
      manyList: [],
      onlyList: [],
      // 上传数据URL
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 设置请求头
      headers: {
        Authorization: window.sessionStorage.getItem('token')
      },
      url: '',
      prevewDialogVisible: false
    }
  },
  created () {
    this.getCateLists()
  },
  methods: {
    async getCateLists () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.$message.success('获取商品分类成功')
      this.getcateList = res.data
      console.log(res)
    },
    handleChange () {
      console.log(this.addForm.goods_cat)
    },
    beforeTabs (activeName, oldActiveName) {
      if (this.addForm.goods_cat.length !== 3 && oldActiveName === '0') {
        this.$message.error('请选择分类列表')
        return false
      }
    },
    async tabClick () {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: 'many' } })
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('获取动态参数失败')
        this.$message.success('获取动态参数成功')
        res.data.forEach((item) => {
          item.attr_vals = item.attr_vals.split(',')
        })
        this.manyList = res.data
        console.log(this.manyList)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: 'only' } })
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('获取静态属性失败')
        this.$message.success('获取静态属性成功')
        this.onlyList = res.data
        console.log(this.onlyList)
      } else if (this.activeIndex === '3') {

      }
    },
    // 文件上传成功预览调用的函数
    handlePreview (file) {
      console.log(file)
      this.url = file.response.data.url
      this.prevewDialogVisible = true
    },
    // 文件移除调用的函数
    handleRemove (file) {
      // 1.获取path路径
      const path = file.response.data.tmp_path
      // 2. 得到索引
      const index = this.addForm.pics.findIndex(item => item.pic === path)
      // 3.将对应索引数据删除
      this.addForm.pics.splice(index, 1)
      console.log(this.addForm)
    },
    // 上传成功
    handleSuccess (response) {
      console.log(response)
      const path = response.data.tmp_path
      const pic = {
        pic: path
      }
      this.addForm.pics.push(pic)
      console.log(this.addForm)
    },
    // 添加操作
    addInfo () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('请填写必要数据')
        // console.log('添加数据')
        const form = _.cloneDeep(this.addForm)
        console.log(form)
        // 遍历参数和属性存储到attrs里面
        this.manyList.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(',')
          }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyList.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })

        form.attrs = this.addForm.attrs
        form.goods_cat = form.goods_cat.join(',')
        console.log(form)
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[this.addForm.goods_cat.length - 1]
      }
      return null
    }
  }
}
</script>
<style lang="less" scoped>
.el-alert {
  margin-bottom: 15px;
}
.el-tabs {
  margin: 15px 0;
}
.el-checkbox{
  margin: 0 5px;
}
.addbtn{
  margin-top: 15px;
}
</style>
