<template>
<el-container>
  <el-header>
      <div>
          <img src="../assets/heima.png" alt="">
          <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="loginOut">退出</el-button>
  </el-header>
  <el-container>
    <el-aside :width="iscollapse?'64px':'200px' ">
     <div class="toggle-button" @click="toggle">|||</div>
     <el-menu background-color="#333744" router
      text-color="#fff"
      active-text-color="#409EFF" unique-opened :collapse="iscollapse" :collapse-transition='false' :default-active='paths'>
      <el-submenu :index="item.id+''" v-for='item in menuList' :key="item.id">
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <el-menu-item :index="'/'+items.path" v-for='items in item.children' :key="items.id" @click="getPath(items.path)">
         <template slot="title">
          <i class="el-icon-menu"></i>
          <span>{{items.authName}}</span>
        </template>
        </el-menu-item>
      </el-submenu>
    </el-menu>
    </el-aside>
    <el-main>
      <router-view/>
    </el-main>
  </el-container>
</el-container>
</template>
<script>
export default {
  data () {
    return {
      menuList: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      iscollapse: false,
      paths: ''
    }
  },
  created () {
    this.getMenuList()
    this.paths = window.sessionStorage.getItem('activePath')
  },
  methods: {
    async getMenuList () {
      const { data: res } = await this.$http.get('/menus')
      if (res.meta.status !== 200) return this.$message.error('获取菜单列表失败')
      this.menuList = res.data
    },
    loginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    toggle () {
      this.iscollapse = !this.iscollapse
    },
    getPath (path) {
      path = '/' + path
      this.paths = path
      window.sessionStorage.setItem('activePath', path)
    }
  }
}
</script>
<style lang="less" scoped>
.el-container{
    height: 100%;
}
.el-header{
    background-color:#373D41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #fff;
    div {
        display: flex;
        align-items: center;
        padding-left: 0px;
      span {
        margin-left: 10px;
        font-size: 20px;
      }
    }
}
.el-aside{
     background-color:#333744;
}
.el-main{
     background-color:#eaedf1;
}
.iconfont{
    margin-right: 10px;
}
.el-menu{
    border: none;
}
.toggle-button{
background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
