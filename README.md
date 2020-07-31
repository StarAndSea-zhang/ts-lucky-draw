# ts-lucky-draw

>基于vue-lucky-draw改动的抽奖奖盘，可设置中心抽奖按钮图片，背景图片，奖品图片，
建议奖品在八个以内，由于改动较大，未对原作者的代码提交pr，原作者启动抽奖前的事件before-start有个小毛病,
在抽奖之前能不断点击按钮发起before-start，且点击非抽奖按钮也能启动抽奖，也就是点击奖盘也能启动before-start，
一般before-start是用来请求抽到的奖品索引，所以请求只能发一次，且也是点击按钮才发送抽奖请求，这部分也有改善。
原作者buuing 的github：https://github.com/buuing/vue-luck-draw
# 项目注意
>- 奖盘按钮背景图片需要自行处理大小，直接按我的图片算的概率，写的很死 
- 奖品名称我限制为不能超过12个字，超过影响奖品图片效果
- 另外奖品图片不因 奖品名称字数减少而自适应大小，奖品图片大小是保持一致的，
背景图片需存在assets/images文件夹下，替换图片可选props传递参数，或直接保持原背景图名替换图片
- 针对图片链接支持绝对路径，相对路径，图片请求错误加载默认图片，绝对路径图片请保证该路径图片服务器开启跨域
  
  出现图片跨域问题，可在该服务nginx上配置
  ~~~
          location / {
              root   html/images;
              index  index.html index.htm;
              add_header Access-Control-Allow-Origin *;
              add_header Access-Control-Allow-Headers X-Requested-With;
              add_header Access-Control-Allow-Methods GET,POST,OPTIONS; 
          }
  ~~~ 

>有问题可邮件2465686113@qq.com
## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

# 使用文档
## 抽奖奖盘
~~~
<template>
    <div id="app">
        <LuckDraw
                ref="luckDraw"
                v-model="currIndex"
                :awards="awards"
                :async="true"
                @before-start="handleBeforeStart"
                @start="handleStart"
                @end="handleEnd"
        />
    </div>
</template>

<script lang="ts">
    import {Component, Vue} from "vue-property-decorator";
    import HelloWorld from "./components/HelloWorld.vue";
    import LuckDraw from "./components/vue-lucky-draw.vue";

    @Component({
        components: {
            HelloWorld,
            LuckDraw
        }
    })
    export default class App extends Vue {
        currIndex = 0
        awards = [     // 奖品 url:'http://merryzoe.cn:8081/award-http.png'
            {name: '0价值5988元华为 P30pro', color: '#f9e3bb',url:'/award-http.png'},
            {name: '1价值398元车载空气净化器', color: '#f8d384'},
            {name: '2价值25元百叶帘遮阳挡', color: '#f9e3bb'},
            {name: '3元油卡套餐红包', color: '#f8d384'},
            {name: '4元油卡直冲红包', color: '#f9e3bb'},
            {name: '5元话费直冲红包', color: '#f8d384'},
            {name: '6价值32元重力感应手机支架', color: '#f9e3bb'},
            {name: '7价值198元手提迷你车在保温冷藏箱', color: '#f8d384'},

        ]

        async handleBeforeStart(e: any) { // 新增异步抽奖
            console.log('开始抽奖前',Math.random());
            let curIndex = 0;
            curIndex = await new Promise((resolve, reject)=> {
                setTimeout(()=>{
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
~~~

## 获奖滚动名单
~~~       
 <rolling-name-list :data="nameList"></rolling-name-list>
~~~
