<template>
    <section class="app-main">
        <el-scrollbar style="height:100%">
            <transition name="fade-transform" mode="out-in">
                <router-view :key="key"/>
            </transition>
            <el-backtop target=".app-main .el-scrollbar__wrap">
                <el-tooltip effect="dark" content="返回顶部" placement="left">
                    <div
                        style="{
        height: 100%;
        width: 100%;
        border-radius:50%;
        background-color: #DCDFE6;
        box-shadow: 0 0 6px rgba(0,0,0, .12);
        text-align: center;
        line-height: 40px;
        color: #1989fa;
      }">
                        <i class="el-icon-caret-top"/>
                    </div>
                </el-tooltip>
            </el-backtop>
        </el-scrollbar>
    </section>
</template>

<script>
    export default {
        name: 'AppMain',
        computed: {
            key() {
                return this.$route.path
            }
        }
    }
</script>

<style lang="scss" scoped>
@import "~@/styles/variables.scss";

.app-main {
    /*107 = navbar 50 + topbar 57  */
    height: 100vh;
    width: 100%;
    position: relative;
    overflow: hidden;
}

.fixed-header + .app-main {
    padding-top: 91px;
}
</style>

<style lang="scss">
// fix css style bug in open el-dialog
.el-popup-parent--hidden {
    .fixed-header {
        padding-right: 15px;
    }
}

</style>
