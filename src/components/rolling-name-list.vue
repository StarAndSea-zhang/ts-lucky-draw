<template>
    <div class="w-rolling">
        <ul v-if="this.data.length!==0" class="w-rolling__list" :class=" {marquee_top:animate}">
            <li v-for="(item) in list" :key="item.id">
                <div class="item">
                    <span class="item-style">{{item.user_name}}</span>
                    <span class="item-style">{{item.phone}}</span>
                    <span class="item-style">{{item.prize_name}}</span>
                    <span class="item-style">{{item.create_time}}</span>
                </div>
            </li>
        </ul>
        <div v-else>
            暂无获奖人员
        </div>
    </div>
</template>

<script>
    //0.7秒
    const animateTime = 700;
    export default {
        name: "rolling-name-list",
        components: {},
        props: {
            data: {
                type: Array,
                default: () => {
                    return []
                }
            }
        },
        data() {
            return {
                //只展示五条数据
                list: this.data.slice(0, 5),
                animateTime: animateTime,
                animate: false,
                currentIndex: 5,
                timer:null
            }
        },
        watch: {
            data(newVal) {
                newVal.forEach(item => {
                    item.create_time = new Date(parseInt(item.create_time) * 1000).toLocaleString();
                    //对号码进行隐藏
                    item.phone = item.phone.substr(0, 3) + "****" + item.phone.substr(7)
                })
                //获奖名单小于5个不进行滚动
                if(newVal.length<5){
                    clearInterval(this.timer);
                }
                this.list = newVal.slice(0, 5);
            }
        },
        mounted() {
            const that = this;
            this.timer = setInterval(() => {
                    if (this.currentIndex > this.data.length - 1) {
                        this.currentIndex = 0;
                    }
                    that.list.push(this.data[this.currentIndex]);
                    this.currentIndex++;
                    that.list.shift();
            }, that.animateTime);
            // 通过$once来监听定时器，在beforeDestroy钩子可以被清除。
            this.$once("hook:beforeDestroy", () => {
                clearInterval(this.timer);
            });
        }
    }
</script>

<style lang="less" scoped>
    @height-rolling-list: 60px;
    @color-hightlight: rgb(255, 81, 30);
    .w-rolling {
        &__list {
            width: 100%;
            margin-top: 10px;
            min-height: @height-rolling-list;
        }

        .van-empty {
            padding: 0;
        }
    }

    .item {
        display: flex;
        padding: 5px 0;
        justify-content: space-between;
    }

    .item-style {
        text-align: center;
        font-size: 12px;
        flex: 1;
    }
</style>
