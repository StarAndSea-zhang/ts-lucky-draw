<!--
 * @Description: 1.基于vue-lucky-draw增加渲染抽奖商品图片的功能
 * @Author npm i vue-lucky-draw
 * @Date 2020/7/21
 -->
<template>
    <canvas id="canvas" class="ldq-luck"></canvas>
</template>

<script>
    export default {
        data() {
            return {
                canvas: '',
                ctx: '',
                startRadian: 0, // 初始角度
                canBeClick: true, // 控制抽奖进行中不让再抽奖
                currIndex: this.value,
                awardImageArr: [],//奖品数组
                btnImage: null,
                backgroundImage: null,
                //12个文字占据的高度
                maxTextLineHeight: 80,
            }
        },
        props: {
            isRunning: {type: Boolean, default: false},          //是否在运行中
            isPreview: {type: Boolean, default: false},          //是否做预览功能
            async: {type: Boolean, default: false},              // 开启异步抽奖
            value: {type: Number, default: 0},                   // 中奖的索引
            awards: {type: Array, default: () =>{return []}},    // 奖品数组
            rate: {type: Number, default: 20},                   // 转盘速率
            radius: {type: Number, default: 200},                // 转盘半径
            textFontSize: {type: String, default: '10px'},       // 文字大小
            lineHeight: {type: Number, default: 20},             // 文字行高
            textColor: {type: String, default: '#d64737'},       // 文字颜色
            textMargin: {type: Number, default: 30},             // 文字距离边框距离
            textPadding: {type: Number, default: 0},             // 文字补偿宽度
            panelImage:{type: String, default: 'background-lucky-panel'},//奖盘外边框背景图片
            defaultAwardImage:{type: String, default: 'icon-award'},//默认奖品图片
            buttonImage:{type: String, default: 'bth-lucky-draw'},//默认按钮背景图
        },
        watch: {
            value: {
                immediate: true,
                deep: true,
                handler(val) {
                    this.currIndex = val
                },
            },
            awards(newVal) {
                this.awards = newVal;
            },
            isPreview: {
                immediate: true,
                deep: true,
                handler(val) {
                    this.isPreview = val;
                },
            }
        },
        computed: {
            //以下参数是根据默认奖盘宽度调整的图片概率，可直接对设计提需求调整图片
            btnWidth() {
                return this.radius * 0.75;
            },
            btnHeight() {
                return this.radius * 0.885
            },
            // 奖品图片宽度
            imageWidth() {
                return this.radius * 0.2
            },
            imageHeight() {
                return this.radius * 0.2
            },
            prizeBlockborder() {
                return this.radius * 0.15
            },
        },
        async mounted() {
            this.initCanvas();
        },
        methods: {

            /**
             * 通过tan求取角度
             */
            getTanDeg(tan) {
                let result = Math.atan(tan) / (Math.PI / 180);
                result = Math.round(result);
                return result;
            },
            /**
             * 初始化转盘
             */
            initCanvas() {
                this.canvas = document.querySelector('#canvas')
                this.ctx = this.canvas.getContext('2d')
                this.canvas.width = this.radius * 2
                this.canvas.height = this.radius * 2
                this.render()
                this.startRotate()
            },
            /**
             * 奖品图片由于异步原因需先初始化,如果服务端返回奖品图片为空值，则需加载默认奖品图片
             */
            loadLocalAwardImage(url) {
                return new Promise((resolve, reject) => {
                    const image = new Image();
                    // images.src = iconSrc;
                    image.src = require('@/assets/images/' + url + '.png');
                    image.crossOrigin = 'Anonymous';
                    image.onload = () => {
                        resolve(image);
                    };
                    image.onerror = () => {
                        reject(image);
                    }
                });
            },
            /**
             * 奖品图片由于异步原因需先初始化，所有的图片画在canvas之前需先初始化，否则有加载不出来的效果
             * canvas存在图片跨域问题
             */
            loadHttpAwardImage(url) {
                return new Promise((resolve, reject) => {
                    const image = new Image();
                    image.src = url;
                    image.crossOrigin = 'anonymous';
                    image.onload = () => {
                        resolve(image);
                    };
                    image.onerror = async () => {
                        // console.log('加载http图片失败')
                        reject(image);
                    }
                });
            },
            /**
             * 不止加载奖品图片，arr[length-1]是固定用来加载开始抽奖按钮图片和arr[length-2]加载面板背景图
             */
            loadAwardImageArr() {
                const promiseArr = []
                this.awards.forEach((item) => {
                    if (item.url) {
                        //如果加载http奖盘失败，加载默认本地图片
                        promiseArr.push(this.loadHttpAwardImage(item.url).then(resolve=>{return resolve},reject=>{
                           return this.loadLocalAwardImage(this.defaultAwardImage);
                        }))
                    } else
                        promiseArr.push(this.loadLocalAwardImage(`icon-award`))
                });
                promiseArr.push(this.loadLocalAwardImage('bth-lucky-draw'))
                promiseArr.push(this.loadLocalAwardImage('background-lucky-panel'))
                return Promise.all(promiseArr)
            },
            /**
             * 处理文字换行
             */
            getLineTextList(ctx, text, maxLineWidth) {
                maxLineWidth += this.textPadding
                const wordList = text.split('');
                let tempLine = '';
                const lineList = []
                for (let i = 0; i < wordList.length; i++) {
                    if (ctx.measureText(tempLine).width >= maxLineWidth) {
                        lineList.push(tempLine)
                        maxLineWidth -= ctx.measureText(text[0]).width
                        tempLine = ''
                    }
                    tempLine += wordList[i]
                }
                lineList.push(tempLine)
                return lineList
            },

            /**
             * 绘制转盘
             */
            drawPanel() {
                const ctx = this.ctx
                ctx.save()
                ctx.drawImage(this.backgroundImage, 0, 0, this.radius * 2, this.radius * 2);
                ctx.restore()
            },
            /**
             * 绘制每个奖品的图片
             * 图片必须居于每个扇叶的中线，所以this.imageWidth/2去比上中线的位置，得到一个开始画图片的坐标与中线的角度
             * 扇叶一半的角度 - 上述角度 = 画笔偏移的坐标角度，从这个角度开始画才可以使图片永远平分在中线两侧
             */
            async drawAwardPic() {
                const ctx = this.ctx;
                const width = this.imageWidth;
                const height = this.imageHeight;
                const awards = this.awards
                let startRadian = this.startRadian
                const RadianGap = Math.PI * 2 / awards.length
                let endRadian = startRadian + RadianGap
                // 画图片的坐标与中线的角度
                const picDeg = this.getTanDeg(this.imageWidth / 2 / (this.radius - this.maxTextLineHeight));
                //picDeg*(Math.PI/180) - 换算角度对应的弧度
                //RadianGapPic - 每张图片偏移的弧度
                const RadianGapPic = RadianGap / 2 - picDeg * (Math.PI / 180);
                for (let i = 0; i < awards.length; i++) {
                    ctx.save()
                    ctx.translate(
                        this.radius + Math.cos(startRadian + RadianGapPic) * (this.radius - this.maxTextLineHeight),
                        this.radius + Math.sin(startRadian + RadianGapPic) * (this.radius - this.maxTextLineHeight)
                    )
                    const rotateNum = startRadian + RadianGap / 2 + Math.PI / 2
                    ctx.rotate(rotateNum)
                    if (this.awardImageArr[i])
                        ctx.drawImage(this.awardImageArr[i], 0, 0, width, height);
                    ctx.restore()
                    startRadian += RadianGap
                    endRadian += RadianGap
                }
            },

            /**
             * 绘制奖品
             */
            drawPrizeBlock() {
                const ctx = this.ctx
                const awards = this.awards
                // 根据初始角度来绘制奖品块
                let startRadian = this.startRadian
                const RadianGap = Math.PI * 2 / awards.length
                let endRadian = startRadian + RadianGap
                for (let i = 0; i < awards.length; i++) {
                    ctx.save()
                    ctx.beginPath()
                    ctx.fillStyle = awards[i].color || (i % 2 == 0 ? '#f8d384' : '#f9e3bb')
                    ctx.moveTo(this.radius, this.radius)
                    ctx.arc(this.radius, this.radius, this.radius - this.prizeBlockborder, startRadian, endRadian, false)
                    ctx.fill()
                    ctx.restore()

                    ctx.save()
                    ctx.fillStyle = this.textColor
                    ctx.font = `${this.textFontSize} Arial`
                    ctx.translate(
                        this.radius + Math.cos(startRadian + RadianGap / 2) * (this.radius - this.textMargin),
                        this.radius + Math.sin(startRadian + RadianGap / 2) * (this.radius - this.textMargin)
                    )
                    ctx.rotate(startRadian + RadianGap / 2 + Math.PI / 2)
                    const awardsName = awards[i].name.slice(0, 12);
                    this.getLineTextList(ctx, awardsName, 70).forEach((line, index) => {
                        //测量切割分段后每个不同的文字宽度
                        const measureText = -ctx.measureText(line).width / 2
                        ctx.fillText(line, measureText, ++index * this.lineHeight)
                    })

                    ctx.restore()
                    startRadian += RadianGap
                    endRadian += RadianGap
                }
            },

            // 将canvas再window中的坐标点转化为canvas中的坐标点
            windowToCanvas(canvas, e) {
                // getBoundingClientRect这个方法返回html元素的大小及其相对于视口的位置
                const canvasPostion = canvas.getBoundingClientRect(), x = e.clientX, y = e.clientY
                return {
                    x: x - canvasPostion.left,
                    y: y - canvasPostion.top
                }
            },
            /**
             * 监听开始游戏的方法
             * @param index
             * @param e
             * @returns {number}
             */
            play(index, e) {
                //如果是预览，不进行抽奖
                if (this.isPreview) return 0;
                if (this.isRunning) return 0;
                //每个扇叶的角度
                const RadianGap = Math.PI * 2 / this.awards.length;
                const dom = document.querySelector('.ldq-luck')
                const canvas = this.canvas
                const ctx = this.ctx
                if (index < 0 || index >= this.awards.length) console.error('该索引的奖品不存在!')
                if (!this.canBeClick || index < 0 || index >= this.awards.length) return false
                this.currIndex = index
                this.canBeClick = false
                const loc = e ? this.windowToCanvas(canvas, e) : { // 模拟点击坐标
                    x: dom.offsetWidth / 2,
                    y: dom.offsetHeight / 2
                }
                ctx.beginPath()
                ctx.arc(this.radius, this.radius, 50, 0, Math.PI * 2, false)
                if (ctx.isPointInPath(loc.x, loc.y)) {
                    this.$emit('start')
                    // 每次点击抽奖，都将初始化角度重置
                    this.startRadian = -Math.floor(Math.random() * 180)
                    // distance是计算出的将指定奖品旋转到指针处需要旋转的角度距离，distanceToStop下面会又说明
                    const distance = this.distanceToStop(this.currIndex)
                    this.rotatePanel(distance)
                } else {
                    this.canBeClick = true
                }
            },
            // 初始化
            startRotate() {
                const canvas = this.canvas
                const ctx = this.ctx
                const canvasStyle = canvas.getAttribute('style');
                this.render()
                //异步抽奖
                if (this.async) {
                    canvas.addEventListener('mousedown', e => {
                        const loc = this.windowToCanvas(canvas, e)
                        ctx.beginPath()
                        ctx.arc(this.radius, this.radius, 30, 0, Math.PI * 2, false)
                        //限定before-start 事件 只在抽奖按钮范围内才执行 传递e值是为了play(index, e)
                        if (ctx.isPointInPath(loc.x, loc.y))
                            this.$emit('before-start', e)
                    })
                } else {
                    canvas.addEventListener('mousedown', e => this.play(this.currIndex, e))
                }
                canvas.addEventListener('mousemove', e => {
                    const loc = this.windowToCanvas(canvas, e)
                    ctx.beginPath()
                    ctx.arc(this.radius, this.radius, 30, 0, Math.PI * 2, false)
                    if (ctx.isPointInPath(loc.x, loc.y)) {
                        canvas.setAttribute('style', `cursor: pointer;${canvasStyle}`)
                    } else {
                        canvas.setAttribute('style', canvasStyle)
                    }
                })
            },

            /**
             * 点击开始后处理面板旋转
             * @param distance
             * @returns {boolean}
             */
            rotatePanel(distance) {
                // 这里用一个很简单的缓动函数来计算每次绘制需要改变的角度，这样可以达到一个转盘从块到慢的渐变的过程
                const changeRadian = (distance - this.startRadian) / this.rate
                // let changeRadian = distance - this.startRadian
                this.startRadian += changeRadian
                // 当最后的目标距离与startRadian之间的差距低于0.05时，就默认奖品抽完了，可以继续抽下一个了。
                if (distance - this.startRadian <= 0.05) {
                    this.$emit('input', this.currIndex)
                    this.$emit('end', this.currIndex)
                    return this.canBeClick = true
                }
                this.render()
                window.requestAnimationFrame(this.rotatePanel.bind(this, distance))
            },

            // 绘制按钮，以及按钮上start的文字
            drawButton() {
                const ctx = this.ctx
                ctx.save();
                ctx.drawImage(this.btnImage, this.radius - this.btnWidth / 2, this.radius - this.btnHeight / 2, this.btnWidth, this.btnHeight);
                ctx.restore();
            },

            distanceToStop(currIndex) {
                // middleDegrees为奖品块的中间角度（最终停留都是以中间角度进行计算的）距离初始的startRadian的距离，distance就是当前奖品跑到指针位置要转动的距离。
                let middleDegrees = 0, distance = 0
                // 映射出每个奖品的middleDegrees
                const awardsToDegreesList = this.awards.map((data, index) => {
                    const awardRadian = (Math.PI * 2) / this.awards.length
                    return awardRadian * index + (awardRadian * (index + 1) - awardRadian * index) / 2
                });
                // 此次抽奖应该中的奖品
                const currentPrizeIndex = currIndex
                middleDegrees = awardsToDegreesList[currentPrizeIndex];
                // 因为指针是垂直向上的，相当坐标系的Math.PI/2,所以这里要进行判断来移动角度
                distance = Math.PI * 3 / 2 - middleDegrees
                distance = distance > 0 ? distance : Math.PI * 2 + distance
                // 这里额外加上后面的值，是为了让转盘多转动几圈，看上去更像是在抽奖
                return distance + Math.PI * 10;
            },

            render() {
                this.loadAwardImageArr().then((list) => {
                    this.awardImageArr = list.slice(0, this.awards.length);
                    // console.log('this.awardImageArr', this.awardImageArr)
                    this.btnImage = list[list.length - 2];
                    this.backgroundImage = list[list.length - 1]
                    this.drawPanel()
                    this.drawPrizeBlock()
                    this.drawButton()
                    //绘制奖品图片
                    this.drawAwardPic();
                })
            }
        }
    }
</script>

