<!--
 * @Description: 动效提供动画json,引入可用的案例 - 后续需删除
 -->
<template>
    <div id="app">
        <lottie :options="defaultOptions" :height="400" :width="400" v-on:animCreated="handleAnimation"/>
        <div>
            <p>Speed: x{{animationSpeed}}</p>
            <input type="range" value="1" min="0" max="3" step="0.5"
                   v-on:change="onSpeedChange" v-model="animationSpeed">
        </div>
        <button v-on:click="stop">stop</button>
        <button v-on:click="pause">pause</button>
        <button v-on:click="play">play</button>
    </div>

</template>

<script>
    import Lottie from './Lottie.vue';
    import * as animationData from '@/common/assets/animation/pinjump.json';

    // import * as animationData from '@/common/assets/animation/data.json';
    //图片路径引入问题
    animationData.assets.forEach((item) => {
        item.u = '';
        if (item.w && item.h) {
            item.p = require(`@/common/assets/animation/images/${item.p}`);
        }
    }); // 获取静态资源
    export default {
        components: {
            'lottie': Lottie
        },
        data() {
            return {
                defaultOptions: {animationData: animationData.default },
                animationSpeed: 1
            }
        },
        methods: {
            handleAnimation: function (anim) {
                this.anim = anim;
            },

            stop: function () {
                this.anim.stop();
            },

            play: function () {
                this.anim.play();
            },

            pause: function () {
                this.anim.pause();
            },

            onSpeedChange: function () {
                this.anim.setSpeed(this.animationSpeed);
            }
        }
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }

    h1, h2 {
        font-weight: normal;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
