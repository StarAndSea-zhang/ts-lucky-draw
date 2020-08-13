<!-- 随机出的标签组件 随机颜色和大小属性
 * @Description:
 * @Author ZhangYu
 * @Date 2020/8/12
 -->
<template>
        <a class="tags" :style="{color:setColor,fontSize:normalSize,fontWeight:600 ,animationDelay:animationDelay}">
            {{content}}
        </a>
</template>

<script>
    import {isColor} from '@/utils/validator';

    export default {
        name: 'RandomTag',
        props: {
            content: String,	//文字内容
            bgColor: {		//背景颜色
                validator(v) {
                    return isColor(v)
                }
            },
            color: {			//文字颜色
                validator(v) {
                    return isColor(v)
                },
            },
            maxFontSize:{
                type:Number,
                default:22
            }
        },
        computed: {
            //随机灰度 黑色 透明度0.2-0.9
            setColor() {
                const randomOpacity = (Math.floor(Math.random() * 7 + 2) / 10).toFixed(1);
                return `rgb(0,0,0,${randomOpacity})`;
            },
            //利用Box-Muller方法极坐标形式    使用两个均匀分布产生一个正态分布
            normalSize() {
                let u = 0.0, v = 0.0, w = 0.0, c = 0.0;
                do {
                    //获得两个（-1,1）的独立随机变量
                    u = Math.random() * 2 - 1.0;
                    v = Math.random() * 2 - 1.0;
                    w = u * u + v * v;
                } while (w == 0.0 || w >= 1.0)
                c = Math.sqrt((-2 * Math.log(w)) / w);
                let size = 12 + u * c * 8;
                //如果随机出的size大于最大字体大小, 按照最大字体处理,默认22px
                if (size > this.maxFontSize) {
                    size = this.maxFontSize;
                }
                return size + 'px';
            },
            /**
             * 随机[min,max]范围
             * @param min 最小数
             * @param max 最大数
             */
            animationDelay(){
                const min = 0;
                const max = 7;
               let second = Math.random()* (max - min)+min;
               return `${second}s`
            }
        }
    }
</script>

<style lang="less" scoped>

    @keyframes fadeio {
        /*设置内容由显示变为隐藏*/
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
    @-moz-keyframes fadeio {
        /*设置内容由显示变为隐藏*/
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
    @-webkit-keyframes fadeio {
        /*设置内容由显示变为隐藏*/
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
    @-o-keyframes fadeio {
        /*设置内容由显示变为隐藏*/
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
    .tags {
        padding: 0 5px;
        font-size: 24px;
        display: inline-block;
        user-select: none;
        animation: fadeio;
        -webkit-animation: fadeio;
        -moz-animation: fadeio;
        -o-animation: fadeio;
        -webkit-animation-duration: 3s; /*动画持续时间*/
        -webkit-animation-iteration-count: infinite; /*动画次数*/
    }
</style>
