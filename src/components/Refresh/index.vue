<template>
    <div class="refresh-container">
        <svg-icon
            icon-class="refresh"
            :class="{ animation: animation }"
            class-name="card-panel-icon"
            @click="refreshSelectedTag"
        />
    </div>
</template>

<script>
    export default {
        name: 'Refresh',
        data() {
            return {
                view: null,
                animation: false,
                redirectLink: ''
            }
        },
        watch: {
            $route(val) {
                if (this.animation) {
                    // eslint-disable-next-line eqeqeq
                    if (val.fullPath.indexOf('/redirect/') != -1) {
                        this.redirectLink = val.path.replace('/redirect', '')
                        // this.animation = false;
                    }
                }
                // eslint-disable-next-line eqeqeq
                if (val.path == this.redirectLink) {
                    this.animation = false
                }
            }
        },
        methods: {
            refreshSelectedTag() {
                // this.animation = false;
                this.animation = true
                let fullPath = this.$route.fullPath
                fullPath = fullPath.replace('/redirect', '')
                this.$router.replace({
                    path: '/redirect' + fullPath
                })
            }
        }
    }
</script>

<style scoped>
    .animation {
        transition: all 0.3s;
        animation: rotate 0.6s linear;
    }

    @keyframes rotate {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    .refresh-container {
        line-height: 56px;
        height: 100%;
        float: left;
        cursor: pointer;
        -webkit-tap-highlight-color: transparent;
        padding: 0 10px;
        fill:currentColor;
        color: #fff;
    }

    .refresh-container svg {
        display: inline-block;
        vertical-align: middle;/*
        width: 20px;
        height: 20px;*/
        font-size: 22px;
        fill:currentColor;
        color: #fff;
    }
</style>
