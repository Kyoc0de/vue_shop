<template>
    <el-container class="home-container">
<!--        头部区域    -->
        <el-header>
            <div>
                <img src="../assets/codelogo.png" alt="" width="60px">
                <span>Seneca 学生网站 BETA 0.1</span>
            </div>
            <span>用户 </span>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
<!--        页面主题区域  -->
        <el-container>
<!--            侧边栏 -->
            <el-aside :width="isCollapse ? '64px':'200px'">
                <div class="toggle-button" @click="toggleCollapse"> ||| </div>
<!--                侧边栏区域   -->
                <el-menu
                        :default-active="activePath"
                        :router="true"
                        :collapse-transition="false"
                        :collapse="isCollapse"
                        :unique-opened="true"
                        background-color="#333744"
                        text-color="#fff"
                        active-text-color="#409eff">
<!--                    一级菜单-->
                    <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
<!--                        一级菜单模板区域-->
                        <template slot="title">
<!--                            图标-->
                            <i class="el-icon-s-tools"></i>
<!--                            文本-->
                            <span>{{item.authName}}</span>
                        </template>
<!--                        二级菜单-->
                        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/'+subItem.path)">
                            <template slot="title">
                                <!--                            图标-->
                                <i class="el-icon-menu"></i>
                                <!--                            文本-->
                                <span>{{subItem.authName}}</span>
                            </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
<!--            右侧主体区域-->
            <el-main>
<!--                路由占位符   -->
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
    export default{
        data(){
            return {
                //左侧菜单
                menulist: [],
                //是否折叠
                isCollapse: false,
                //被激活的连接地址
                activePath: ''
            }
        },
        created() {
            this.getMenuList()
            this.activePath = window.sessionStorage.getItem('activePath')
        },
        methods: {
            logout(){
                window.sessionStorage.clear()
                this.$router.push('/login')
            },
            //获取所有菜单
            async getMenuList(){
                const { data: res} = await this.$http.get('menus')
                if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
                this.menulist = res.data
                console.log(res);
            },
            //点击按钮,切换展开
            toggleCollapse(){
                this.isCollapse = ! this.isCollapse
            },
            //保存连接激活信息
            saveNavState(activePath){
                window.sessionStorage.setItem('activePath',activePath)
                this.activePath = activePath
            }
        }
    }
</script>

<style lang="less" scoped>

    .home-container{
        height: 100%;
    }
    .el-header{
        background-color: #373D41;
        display: flex;
        justify-content: space-between;
        padding-left: 0;
        align-items: center;
        color: #ffffff;
        font-size: 20px;
        > div {
            display: flex;
            align-items: center;
            span {
                margin-left: 15px;
            }
        }
    }

    .el-aside{
        background-color: #333744;
        .el-menu{
            border-right: none;
        }
    }

    .ek-main{
        background-color: #eaedf1;
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