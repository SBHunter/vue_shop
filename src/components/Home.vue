<template>
  <el-container class="home-container">
    <!--头部区域-->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button></el-header>
    <!--页面主题区域-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="elAsideWidth">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
                background-color="#333744"
                text-color="#fff"
                active-text-color="#409eff"
                :unique-opened="true"
                :collapse="isCollapse"
                :collapse-transition="false"
                router
                :default-active="activePath"
        >
          <el-submenu :index="item.path + ''" v-for="item of menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]" class="iconfont"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/' + child.path" v-for="child of item.children" :key="child.id"
                          @click="saveNavState('/' + child.path)">
              <i class="el-icon-menu"></i>
              {{child.authName}}
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!--右侧内容主题-->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    data() {
      return {
        //左侧菜单数据
        menuList:[],
        iconsObj: {
          125: 'iconfont icon-users',
          103: 'iconfont icon-tijikongjian',
          101: 'iconfont icon-shangpin',
          102: 'iconfont icon-danju',
          145: 'iconfont icon-baobiao'
        },
        isCollapse:false,
        elAsideWidth: '200px',
        //被激活的地址
        activePath: ''
      }
    },
    created() {
      this.getMenuList();
      this.activePath = window.sessionStorage.getItem('activePath');
    },
    methods: {
      logout() {
        window.sessionStorage.clear()
        this.$router.push('/login')
      },
      async getMenuList() {
        const {data:res} = await this.$http.get('menus');
        if(res.meta.status !== 200) return this.$message.error(res.meta.msg);
        this.menuList = res.data;
      },
      toggleCollapse() {
        this.isCollapse = ! this.isCollapse;
        if(this.isCollapse)
          this.elAsideWidth = '64px';
        else
          this.elAsideWidth = '200px';
      },
      saveNavState(activePath) {
        window.sessionStorage.setItem('activePath', activePath)
        this.activePath = activePath;
      }
    }
  }
</script>

<style lang="less" scoped>
  .el-header{
    background-color: #373D41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    div{
      display: flex;
      align-items: center;
      span {
        padding: 15px;
      }
    }
  }

  .el-aside{
    background-color: #333744;
    .el-menu{
      border-right: none;
    }
  }

  .el-main{
    background-color: #eaedf1;
  }

  .home-container{
    width: 100%;
    height: 100%;
  }

  .iconfont{
    margin-right: 10px;
    align-content: center;
  }

  .toggle-button{
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: white;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;

  }

</style>
