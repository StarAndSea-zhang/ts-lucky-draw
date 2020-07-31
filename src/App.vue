<template>
    <div id="app">
        <!--        抽奖奖盘-->
        <luck-draw
                ref="luckDraw"
                v-model="currIndex"
                :awards="awards"
                :async="true"
                @before-start="handleBeforeStart"
                @start="handleStart"
                @end="handleEnd"
        />
        <!--滚动名单-->
        <rolling-name-list :data="nameList"></rolling-name-list>
    </div>
</template>

<script lang="ts">
    import {Component, Vue} from "vue-property-decorator";
    import HelloWorld from "./components/hello-world.vue";
    import LuckDraw from "./components/vue-lucky-draw.vue";
    import RollingNameList from './components/rolling-name-list.vue'

    @Component({
        components: {
            HelloWorld,
            LuckDraw,
            RollingNameList
        }
    })
    export default class App extends Vue {
        currIndex = 0
        awards = [     // 奖品 url:'http://merryzoe.cn:8081/award-http.png'
            {name: '0价值5988元华为 P30pro', color: '#f9e3bb', url: '/award-http.png'},
            {name: '1价值398元车载空气净化器', color: '#f8d384'},
            {name: '2价值25元百叶帘遮阳挡', color: '#f9e3bb'},
            {name: '3元油卡套餐红包', color: '#f8d384'},
            {name: '4元油卡直冲红包', color: '#f9e3bb'},
            {name: '5元话费直冲红包', color: '#f8d384', url: 'http://merryzoe.cn:8081/award-absolute-path.png'},
            {name: '6价值32元重力感应手机支架', color: '#f9e3bb'},
            {name: '7价值198元手提迷你车在保温冷藏箱', color: '#f8d384'},

        ]

        nameList = [{
            userName: '哇哇',
            phone: '17287654324',
            prizeName: '火箭',
            createTime: '1596175677'
        }, {
            userName: '楠楠',
            phone: '17287654111',
            prizeName: '汽车',
            createTime: '1596186477'
        }, {
            userName: '逸明',
            phone: '17287654222',
            prizeName: '五三交学',
            createTime: '1596986477'
        }, {
            userName: '小橘',
            phone: '17287654333',
            prizeName: '彩电',
            createTime: '1596166477'
        }, {
            userName: '穗穗',
            phone: '17287654333',
            prizeName: '二十个柯柯',
            createTime: '1596175677'
        }, {
            userName: '平安',
            phone: '17287654333',
            prizeName: '一个彩蛋',
            createTime: '1592186477'
        }, {
            userName: '七七',
            phone: '17287658333',
            prizeName: '谢谢参与',
            createTime: '1592180477'
        }];

        async handleBeforeStart(e: any) { // 新增异步抽奖
            console.log('开始抽奖前', Math.random());
            let curIndex = 0;
            curIndex = await new Promise((resolve, reject) => {
                setTimeout(() => {
                    resolve(Math.round(Math.random() * 7 + 0));
                })
            });
            console.log('curIndex', curIndex);
            (this.$refs.luckDraw as any).play(curIndex);

        }

        handleStart() {
            console.log('开始抽奖')
        }

        handleEnd(index: number) {
            // alert('恭喜您抽到大奖, 奖品为' + this.awards[this.currIndex].name)
        }
    }
</script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
