<template>
    <div
        class="cf-drawer-panel-container"
        v-show="visible">
        <div class="mask" @click="close"></div>
        <section :class="['cf-drawer-panel',entered ? 'cf-drawer-open':'', customClass]" :style="computdStyle">
            <header class="cf-drawer-header" v-if="withHeader" >{{ title }}
                <small class="text-tips" v-if="subTitle">
                    &nbsp;&nbsp;&nbsp;&nbsp;{{ subTitle }}
                </small>
                <span class="right">
                    <slot name="head"></slot>
                    <span class="cf-icon cf-close-bold" v-if="showClose" @click="close"></span>
                </span>
            </header>
            <section class="cf-drawer-body" v-if="rendered">
                <slot></slot>
            </section>
        </section>
    </div>
</template>

<script>
    export default {
        name: 'Drawer',

        componentName: 'Drawer',

        props: {

            // 是否显示标题
            withHeader: {
                type: Boolean,
                default: true
            },

            // 宽度
            size: {
                type: String,
                default: '540px'
            },

            // 是否显示关闭按钮
            showClose: {
                type: Boolean,
                default: true
            },

            // 是否在关闭时销毁子组件
            destroyOnClose: {
                type: Boolean,
                default: false
            },

            // 标题
            title: {
                type: String,
                default: ''
            },
            // 自定义样式类
            customClass: {
                type: String,
                default: ''
            },

            // 子标题
            subTitle: {
                type: String,
                default: ''
            },

            // 用于支持在任何地方打开
            appendToBody: {
                type: Boolean,
                default: true
            },

            // 支持指定容器
            container: {
                type: Object,
                default: null
            },

            // 是否显示 需要使用 visible.sync 修饰符，
            visible: {
                type: Boolean
            },

            // 关闭之前要执行的函数 可用来操控抽屉的关闭
            /*
            * @ep
            <drawer :beforeClose="beforeClose"></drawer>
            beforeClose (done) {
                setTimeout(() => {
                    done(true)
                }, 2000)
            }
            */
            beforeClose: {
                type: Function
            }
        },

        data () {
            return {
                entered: false,
                closed: false,
                rendered: false
            }
        },

        watch: {
            visible (val) {
                if (val) {
                    console.log('appendToContainer')
                    this.appendToContainer()
                    this.rendered = true
                    this.open()
                } else {
                    this.close()
                }
            }
        },

        mounted () {
            if (this.visible) {
                this.appendToContainer()
                this.rendered = true
                this.open()
            }
        },

        computed: {
            computdStyle () {
                let { size, entered } = this
                return {
                    width: size,
                    right: entered ? 0 : `-${size}`
                }
            }
        },

        destroyed () {
            if (this.appendToBody && this.$el && this.$el.parentNode) {
                this.$el && this.$el.parentNode && this.$el.parentNode.removeChild(this.$el)
            }
        },

        methods: {

            appendToContainer () {
                if (this.appendToBody) {
                    document.body.appendChild(this.$el)
                } else if (this.container) { // 指定容器
                    this.container.appendChild(this.$el)
                }
            },

            open () {
                this.$emit('open')
                setTimeout(() => {
                    this.entered = true
                    this.closed = false
                }, 100)
            },

            hide (cancel) {
                if (this.closed) return
                if (cancel !== false) {
                    this.entered = false
                    // 一定要延迟哦 否则会直接闪没的
                    setTimeout(() => {
                        this.$emit('update:visible', false)
                        this.$emit('close')
                        if (this.destroyOnClose === true) {
                            this.rendered = false
                        }
                        this.closed = true
                    }, 500)
                }
            },

            closeDrawer () {
                if (typeof this.beforeClose === 'function') {
                    this.beforeClose(this.hide)
                } else {
                    this.hide()
                }
            },

            close () {
                this.closeDrawer()
            }
        }
    }
</script>

<style lang="less">
    .cf-drawer-panel-container {
        .mask {
            z-index: 15;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: transparent;
        }
    }

    .cf-drawer-panel {
        z-index:16;
        position: fixed;
        top: 0;
        right: -100%;
        bottom: 0;
        background-color: #fff;
        box-shadow: 0 5px 12px 0 rgba(49,114,149,0.20);
        transition: right .5s;
        .cf-drawer-header {
            padding: 15px 20px;
            font-size: 16px;
            height: 60px;
            line-height: 30px;
            border-bottom: 1px solid #E1EAF1;
            .cf-close-bold {
                font-size: 12px;
                color: #C2CBD0;
                padding: 4px 10px;
                cursor: pointer;
                &:hover {
                    color: #646b6e;
                }
            }
        }
    }
</style>

