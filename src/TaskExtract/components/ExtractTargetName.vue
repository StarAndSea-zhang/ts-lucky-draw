<!--
 * @Description: 显示随机大小颜色的标签
 * @Author ZhangYu
 * @Date 2020/8/12
 -->
<template>
    <!--    <canvas id="extract-target" class="target-panel"></canvas>-->
    <div class="w-target">
        <div class="w-target__tags">
            <div v-for="(item,index) in columns" :key="index" class="w-target__tags--li">
                    <RandomTag v-for="(tag) in item" :key="tag.id" :content="tag.name"></RandomTag>
            </div>
        </div>
    </div>
</template>

<script>

    import RandomTag from './RandomTag'

    export default {
        name: 'ExtractTargetName',
        components: {
            RandomTag
        },
        props: {
            /**
             *  item 表示每行多少列 length表示多少行
             *  默认为 3行 10列
             */
            rowConfig: {
                type: Array,
                default: () => {
                    return [10, 10, 10]
                }
            }
        },
        data() {
            return {
                list: [],
                tagList: [],
            }
        },
        computed: {
            columns() {
                let temp = this.tagList.sort(function () {    //重新随机排序
                    return Math.random() > .5 ? -1 : 1;
                }).concat();
                return this.rowConfig.map((v, k) => {//根据list生成数据结构
                    return temp.splice(0, v);
                })
            }
        },
        methods: {
            async getTags(next) {//获取tag列表
                try {
                    // let data  = await axios.get(this.$store.state.baseUrl+'api/tags');
                    let data = {
                        "tags": [{"id": 1, "name": "Javascript"}, {"id": 2, "name": "CSS"}, {
                            "id": 3,
                            "name": "数据结构"
                        }, {"id": 4, "name": "HTML"}, {"id": 5, "name": "vue"}, {"id": 6, "name": "react"}, {
                            "id": 7,
                            "name": "reactNative"
                        }, {"id": 8, "name": "Babel"}, {"id": 9, "name": "MVC"}, {"id": 10, "name": "Redux"}, {
                            "id": 11,
                            "name": "Sass"
                        }, {"id": 12, "name": "Less"}, {"id": 13, "name": "HTTP/TCP"}, {
                            "id": 14,
                            "name": "node"
                        }, {"id": 15, "name": "koa"}, {"id": 16, "name": "web开发"}, {"id": 17, "name": "安全"}, {
                            "id": 18,
                            "name": "性能"
                        }, {"id": 19, "name": "算法"}, {"id": 20, "name": "设计模式"}, {"id": 21, "name": "移动端"}, {
                            "id": 22,
                            "name": "函数式编程"
                        }, {"id": 23, "name": "浏览器"}, {"id": 24, "name": "面向对象编程"}, {"id": 25, "name": "缓存"}, {
                            "id": 26,
                            "name": "MySQL"
                        }, {"id": 27, "name": "web"}, {"id": 28, "name": "路由"}, {
                            "id": 29,
                            "name": "Nuxt.js"
                        }, {"id": 30, "name": "WebGL"}, {"id": 31, "name": "Three.js"}, {
                            "id": 32,
                            "name": "代码规范"
                        }, {"id": 33, "name": "TypeScript"}, {"id": 34, "name": "MVVM"}, {
                            "id": 35,
                            "name": "JQuery"
                        }, {"id": 36, "name": "Webpack"}, {"id": 37, "name": "Git"}, {
                            "id": 38,
                            "name": "Gulp"
                        }, {"id": 39, "name": "代码规范"}, {"id": 40, "name": "单元测试"}, {
                            "id": 41,
                            "name": "Canvas"
                        }, {"id": 42, "name": "Nginx"}]
                    };
                    this.tagList = data.tags;
                } catch (err) {
                    console.log(err)
                }
            },
        },
        created() {
            this.getTags();
        }
    }
</script>
<style lang="less" scoped>
    .w-target {
        display: inline-block;
        &__tags {
            text-align: center;
            word-break: break-word;
            &--li {
                display: flex;
                text-align: center;
            }
        }
    }
</style>
