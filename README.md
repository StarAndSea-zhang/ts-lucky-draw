# ts-lucky-draw

>基于vue-lucky-draw改动的抽奖奖盘，可设置中心抽奖按钮图片，背景图片，奖品图片，
建议奖品在八个以内，由于改动较大，未对原作者的代码提交pr，原作者启动抽奖前的事件before-start有个小毛病,
在抽奖之前能不断点击按钮发起before-start，且点击非抽奖按钮也能启动抽奖，也就是点击奖盘也能启动before-start，
一般before-start是用来请求抽到的奖品索引，所以请求只能发一次，且也是点击按钮才发送抽奖请求，这部分也有改善。
原作者buuing 的github：https://github.com/buuing/vue-luck-draw
# 项目注意
奖盘按钮背景图片需要自行处理大小，直接按我的图片算的概率，写的很死 
奖品名称我限制为不能超过12个字，超过影响奖品图片效果
另外奖品图片不因 奖品名称字数减少而自适应大小，奖品图片大小是保持一致的，
背景图片需存在assets/images文件夹下，替换图片可选props传递参数，或直接保持原背景图名替换图片
有问题可邮件2465686113@qq.com
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
