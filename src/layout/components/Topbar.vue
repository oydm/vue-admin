<template>
    <div class="top-nav">
        <div class="log">
            <hamburger v-if="!sidebar.hide" :is-active="sidebar.opened" class="hamburger-container"
                       @toggleClick="toggleSideBar"/>
        </div>
        <el-menu
            :active-text-color="variables.menuActiveText"
            :default-active="activeMenu"
            mode="horizontal"
            @select="handleSelect"
        >
            <div v-for="item in routes" :key="item.path" class="nav-item">
                <app-link :to="resolvePath(item)">
                    <el-menu-item
                        v-if="!item.hidden"
                        :index="item.path"
                    >{{ item.meta ? item.meta.title : item.children[0].meta.title }}
                    </el-menu-item>
                </app-link>
            </div>
        </el-menu>

        <div class="right-menu">
            <template v-if="device!=='mobile'">
                <el-tooltip content="刷新页面" effect="dark" placement="bottom">
                    <Refresh />
                </el-tooltip>
                <search id="header-search" class="right-menu-item"/>
                <screenfull id="screenfull" class="right-menu-item hover-effect"/>
                <el-tooltip content="布局大小" effect="dark" placement="bottom">
                    <size-select id="size-select" class="right-menu-item hover-effect"/>
                </el-tooltip>
            </template>
            <el-dropdown class="avatar-container" trigger="click">

                <div class="avatar-wrapper">
                    <el-avatar :size="40" :src="avatar+'?imageView2/1/w/80/h/80'">
                    </el-avatar>
                    <!--<img :src="avatar+'?imageView2/1/w/80/h/80'" class="user-avatar">-->
                    <i class="el-icon-caret-bottom"/>
                </div>


                <el-dropdown-menu slot="dropdown" class="user-dropdown">
                    <router-link to="/">
                        <el-dropdown-item>主页</el-dropdown-item>
                    </router-link>
                    <!--<a href="https://github.com/PanJiaChen/vue-admin-template/" target="_blank">
                        <el-dropdown-item>Github</el-dropdown-item>
                    </a>
                    <a href="https://panjiachen.github.io/vue-element-admin-site/#/" target="_blank">
                        <el-dropdown-item>Docs</el-dropdown-item>
                    </a>-->
                    <el-dropdown-item divided @click.native="logout">
                        <span style="display:block;">退出</span>
                    </el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </div>
    </div>
</template>

<script>
    import { mapGetters } from 'vuex'
    import AppLink from './Sidebar/Link'
    import { constantRoutes } from '@/router'
    import variables from '@/styles/variables.scss'
    import { isExternal } from '@/utils/validate'
    import Hamburger from '@/components/Hamburger'
    import Screenfull from '@/components/Screenfull'
    import SizeSelect from '@/components/SizeSelect'
    import Search from '@/components/HeaderSearch'
    import Refresh from '@/components/Refresh'

    export default {
        name: 'Topbar',
        components: {
            AppLink,
            Hamburger,
            Screenfull,
            SizeSelect,
            Search,
            Refresh
        },
        data() {
            return {
                routes: constantRoutes
            }
        },
        computed: {
            activeMenu() {
                const route = this.$route
                const { meta, path } = route
                // if set path, the sidebar will highlight the path you set
                if (meta.activeMenu) {
                    return meta.activeMenu
                }
                // 如果是首页，首页高亮
                if (path === '/dashboard') {
                    return '/'
                }
                // 如果不是首页，高亮一级菜单
                const activeMenu = '/' + path.split('/')[1]
                return activeMenu
            },
            variables() {
                return variables
            },
            sidebar() {
                return this.$store.state.app.sidebar
            },
            ...mapGetters(['sidebar',
                           'avatar',
                           'device'])
        },
        mounted() {
            this.initCurrentRoutes()
        },
        methods: {
            toggleSideBar() {
                this.$store.dispatch('app/toggleSideBar')
            },
            // 通过当前路径找到二级菜单对应项，存到store，用来渲染左侧菜单
            initCurrentRoutes() {
                const { path } = this.$route
                let route = this.routes.find(
                    item => item.path === '/' + path.split('/')[1]
                )
                // 如果找不到这个路由，说明是首页
                if (!route) {
                    route = this.routes.find(item => item.path === '/')
                }
                this.$store.commit('permission/SET_CURRENT_ROUTES', route)
                this.setSidebarHide(route)
            },
            // 判断该路由是否只有一个子项或者没有子项，如果是，则在一级菜单添加跳转路由
            isOnlyOneChild(item) {
                if (item.children && item.children.length === 1) {
                    return true
                }
                return false
            },
            resolvePath(item) {
                // 如果是个完成的url直接返回
                if (isExternal(item.path)) {
                    return item.path
                }
                // 如果是首页，就返回重定向路由
                if (item.path === '/') {
                    const path = item.redirect
                    return path
                }

                // 如果有子项，默认跳转第一个子项路由
                let path = ''
                /**
                 * item 路由子项
                 * parent 路由父项
                 */
                const getDefaultPath = (item, parent) => {
                    // 如果path是个外部链接（不建议），直接返回链接，存在个问题：如果是外部链接点击跳转后当前页内容还是上一个路由内容
                    if (isExternal(item.path)) {
                        path = item.path
                        return
                    }
                    // 第一次需要父项路由拼接，所以只是第一个传parent
                    if (parent) {
                        path += (parent.path + '/' + item.path)
                    } else {
                        path += ('/' + item.path)
                    }
                    // 如果还有子项，继续递归
                    if (item.children) {
                        getDefaultPath(item.children[0])
                    }
                }

                if (item.children) {
                    getDefaultPath(item.children[0], item)
                    return path
                }

                return item.path
            },
            handleSelect(key, keyPath) {
                // 把选中路由的子路由保存store
                const route = this.routes.find(item => item.path === key)
                this.$store.commit('permission/SET_CURRENT_ROUTES', route)
                this.setSidebarHide(route)
            },
            // 设置侧边栏的显示和隐藏
            setSidebarHide(route) {
                /*if (!route.children || route.children.length === 1) {
                    this.$store.dispatch('app/toggleSideBarHide', true)
                } else {
                    this.$store.dispatch('app/toggleSideBarHide', false)
                }*/
            },
            async logout() {
                await this.$store.dispatch('user/logout')
                this.$router.push(`/login?redirect=${this.$route.fullPath}`)
            }
        }
    }
</script>
<style lang="scss" scoped>
.log {

    //height: 106px;
    /*overflow: hidden;
    position: relative;
    background: #fff;
    box-shadow: 0 1px 4px rgba(0, 21, 41, .08);
    */
    .hamburger-container {
        line-height: 53px;
        height: 100%;
        float: left;
        cursor: pointer;
        transition: background .3s;
        color: #fff;
        -webkit-tap-highlight-color: transparent;

        &:hover {
            background: rgba(0, 0, 0, .025)
        }
    }
}

.errLog-container {
    display: inline-block;
    vertical-align: top;
}

.right-menu {
    float: right;
    height: 100%;
    line-height: 56px;

    &:focus {
        outline: none;
    }

    .right-menu-item {
        display: inline-block;
        padding: 0 8px;
        height: 100%;
        font-size: 18px;
        color: #fff;
        vertical-align: text-bottom;

        &.hover-effect {
            cursor: pointer;
            transition: background .3s;

            &:hover {
                background: rgba(0, 0, 0, .025)
            }
        }
    }

    .avatar-container {
        margin-right: 30px;
        float: right;
        line-height: 30px;
        .avatar-wrapper {
            margin-top: 5px;
            position: relative;

            .user-avatar {
                cursor: pointer;
                width: 40px;
                height: 40px;
                border-radius: 10px;
            }

            .el-icon-caret-bottom {
                cursor: pointer;
                position: absolute;
                right: -20px;
                top: 25px;
                font-size: 12px;
            }
        }
    }
}
</style>
