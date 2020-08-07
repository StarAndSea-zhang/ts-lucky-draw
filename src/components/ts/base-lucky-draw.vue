<!--
 * @Description:
 * @Author ZhangYu
 * @Date 2020/7/31
 -->
<template>
    <div>
        <canvas id="canvas" class="ldq-luck"></canvas>
    </div>
</template>

<script lang="ts">
    import {Component, Vue, Prop, Watch} from "vue-property-decorator";
    import {IAward} from './types/IAward';

    @Component
    export default class LuckyDraw extends Vue {
        canvas!: HTMLCanvasElement;

        ctx: any | null = null;
        startRadian = 0 // 初始角度
        canBeClick = true // 控制抽奖进行中不让再抽奖
        currIndex = 0;
        awardImageArr: Array<any> = []//奖品数组
        btnImage = null
        backgroundImage = null
        //12个文字占据的高度
        maxTextLineHeight = 80
        mAwards: Array<IAward> = [];

        //是否在运行中
        @Prop({type: Boolean, default: false})
        private isRunning!: boolean

        //是否做预览功能
        @Prop({type: Boolean, default: false})
        private isPreview !: boolean

        // 开启异步抽奖
        @Prop({type: Boolean, default: false})
        async !: boolean

        // 中奖的索引
        @Prop({type: Number, default: 0})
        value = 0;

        // 奖品数组
        @Prop({
            type: Array, default: () => {
                return []
            }
        })
        private awards!: Array<IAward>;

        // 转盘速率
        @Prop({type: Number, default: 20})
        private rate !: number;

        // 转盘半径
        @Prop({type: Number, default: 200})
        private radius !: number;

        // 文字大小
        @Prop({type: String, default: '10px'})
        private textFontSize !: string;

        // 文字行高
        @Prop({type: Number, default: 20})
        private lineHeight !: number;

        // 文字颜色
        @Prop({type: String, default: '#d64737'})
        private textColor !: string;

        // 文字距离边框距离
        @Prop({type: Number, default: 30})
        private textMargin !: number;

        // 文字补偿宽度
        @Prop({type: Number, default: 0})
        private textPadding !: number;

        //奖盘外边框背景图片
        @Prop({type: String, default: 'background-lucky-panel'})
        private panelImage !: string

        //默认奖品图片
        @Prop({type: String, default: 'icon-award'})
        private defaultAwardImage !: string

        //默认按钮背景图
        @Prop({type: String, default: 'bth-lucky-draw'})
        private buttonImage !: string

        @Watch('isPreview', {
            immediate: true,
            deep: true,
        })
        handlePreviewChange(val: boolean) {
            this.isPreview = val;
        }

        @Watch('value', {
            immediate: true,
            deep: true,
        })
        handleValueChange(val: number) {
            this.currIndex = val
        }

        @Watch('awards', {
            immediate: true,
            deep: true,
        })
        handleAwardsChange(newVal: Array<IAward>) {
            this.mAwards = newVal;
        }

        get btnWidth() {
            return this.radius * 0.75;
        }

        get btnHeight() {
            return this.radius * 0.885
        }

        // 奖品图片宽度
        get imageWidth() {
            return this.radius * 0.2
        }

        get imageHeight() {
            return this.radius * 0.2
        }

        get prizeBlockborder() {
            return this.radius * 0.15
        }

        async mounted() {
            this.initCanvas();
        }

        /**
         * 通过tan求取角度
         */
        getTanDeg(tan: number) {
            let result = Math.atan(tan) / (Math.PI / 180);
            result = Math.round(result);
            return result;
        }

        /**
         * 初始化转盘
         */
        async initCanvas() {
            this.canvas = (document.querySelector('#canvas') as HTMLCanvasElement);
            this.ctx = this.canvas!.getContext('2d')
            this.canvas.width = this.radius * 2
            this.canvas.height = this.radius * 2
            await this.beforeRender();
            this.renderLuckDraw();
            this.startRotate()
        }

        /**
         * 加载完图片
         */
        beforeRender() {
            return this.loadAwardImageArr().then((list: Array<any>) => {
                this.awardImageArr = list.slice(0, this.mAwards.length);
                this.btnImage = list[list.length - 2];
                this.backgroundImage = list[list.length - 1]
            })
        }

        /**
         * 注意两个问题：
         * 奖盘组合绘制的顺序不能变，drawPanel画外层边框在层级最下面，最先画，奖品面板，按钮，奖品
         * 转动的效果是靠不断绘制图片的，所以耗性能很大
         */
        renderLuckDraw() {
            //奖盘转动时，可以不画最外围边框
            if (this.canBeClick)
                this.drawPanel()
            this.drawPrizeBlock()
            this.drawButton()
            //绘制奖品图片
            this.drawAwardPic();
        }

        /**
         * 不止加载奖品图片，arr[length-1]是固定用来加载开始抽奖按钮图片和arr[length-2]加载面板背景图
         */
        loadAwardImageArr() {
            const promiseArr = []
            this.mAwards.forEach((item) => {
                if (item.url) {
                    //如果加载http奖盘失败，加载默认本地图片
                    promiseArr.push(this.loadHttpAwardImage(item.url).then(resolve => {
                        return resolve
                    }, (reject: any) => {
                        return this.loadLocalAwardImage(this.defaultAwardImage);
                    }))
                } else
                    promiseArr.push(this.loadLocalAwardImage(`icon-award`))
            });
            promiseArr.push(this.loadLocalAwardImage('bth-lucky-draw'))
            promiseArr.push(this.loadLocalAwardImage('background-lucky-panel'))
            return Promise.all(promiseArr)
        }

        /**
         * 奖品图片由于异步原因需先初始化,如果服务端返回奖品图片为空值，则需加载默认奖品图片
         */
        loadLocalAwardImage(url: string) {
            return new Promise((resolve, reject) => {
                const image = new Image();
                image.src = require('@/assets/images/' + url + '.png');
                image.crossOrigin = 'Anonymous';
                image.onload = () => {
                    resolve(image);
                };
                image.onerror = () => {
                    reject(image);
                }
            });
        }

        /**
         * 奖品图片由于异步原因需先初始化，所有的图片画在canvas之前需先初始化，否则有加载不出来的效果
         * canvas存在图片跨域问题
         */
        loadHttpAwardImage(url: string) {
            return new Promise((resolve, reject) => {
                const image = new Image();
                image.crossOrigin = 'anonymous';
                image.src = url;
                image.onload = () => {
                    resolve(image);
                };
                image.onerror = async () => {
                    // console.log('加载http图片失败')
                    reject(image);
                }
            });
        }

        startRotate() {
            const canvas = this.canvas
            const ctx = this.ctx
            const canvasStyle = canvas.getAttribute('style')!;
            // this.beforeRender();
            this.renderLuckDraw()
            //异步抽奖
            if (this.async) {
                canvas.addEventListener('mousedown', (e: MouseEvent) => {
                    const loc = this.windowToCanvas(canvas, e)
                    ctx.beginPath()
                    ctx.arc(this.radius, this.radius, 30, 0, Math.PI * 2, false)
                    //限定before-start 事件 只在抽奖按钮范围内才执行 传递e值是为了play(index, e)
                    if (ctx.isPointInPath(loc.x, loc.y))
                        this.$emit('before-start', e)
                })
            } else {
                canvas.addEventListener('mousedown', (e: MouseEvent) => this.play(this.currIndex, e))
            }
            canvas.addEventListener('mousemove', (e: MouseEvent) => {
                const loc = this.windowToCanvas(canvas, e)
                ctx.beginPath()
                ctx.arc(this.radius, this.radius, 30, 0, Math.PI * 2, false)
                if (ctx.isPointInPath(loc.x, loc.y)) {
                    canvas.setAttribute('style', `cursor: pointer;${canvasStyle}`)
                } else {
                    canvas.setAttribute('style', canvasStyle)
                }
            })
        }

        /**
         * 将canvas再window中的坐标点转化为canvas中的坐标点
         */
        windowToCanvas(canvas: HTMLCanvasElement, e: MouseEvent) {
            // getBoundingClientRect这个方法返回html元素的大小及其相对于视口的位置
            const canvasPostion = canvas.getBoundingClientRect(), x = e.clientX, y = e.clientY
            return {
                x: x - canvasPostion.left,
                y: y - canvasPostion.top
            }
        }

        /**
         * 监听开始游戏的方法
         * @param index
         * @param e
         * @returns {number}
         */
        play(index: number, e: MouseEvent) {
            //如果是预览，不进行抽奖
            if (this.isPreview) return 0;
            if (this.isRunning) return 0;
            //每个扇叶的角度
            const dom: HTMLElement | null = document.querySelector('.ldq-luck')
            const canvas = this.canvas
            const ctx = this.ctx
            if (index < 0 || index >= this.mAwards.length) console.error('该索引的奖品不存在!')
            if (!this.canBeClick || index < 0 || index >= this.mAwards.length) return false
            this.currIndex = index
            this.canBeClick = false
            const loc = e ? this.windowToCanvas(canvas, e) : { // 模拟点击坐标
                x: (dom as HTMLElement).offsetWidth / 2,
                y: (dom as HTMLElement).offsetHeight / 2
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
        }

        /**
         * 当前
         */
        distanceToStop(currIndex: number) {
            // middleDegrees为奖品块的中间角度（最终停留都是以中间角度进行计算的）距离初始的startRadian的距离，distance就是当前奖品跑到指针位置要转动的距离。
            let middleDegrees = 0, distance = 0
            // 映射出每个奖品的middleDegrees
            const awardsToDegreesList = this.mAwards.map((data, index) => {
                const awardRadian = (Math.PI * 2) / this.mAwards.length
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
        }

        /**
         * 点击开始后处理面板旋转
         * @param distance
         * @returns {boolean}
         */
        rotatePanel(distance: number) {
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
            this.renderLuckDraw()
            window.requestAnimationFrame(this.rotatePanel.bind(this, distance))
        }


        // 绘制按钮，以及按钮上start的文字
        drawButton() {
            const ctx = this.ctx
            ctx.save();
            ctx.drawImage(this.btnImage, this.radius - this.btnWidth / 2, this.radius - this.btnHeight / 2, this.btnWidth, this.btnHeight);
            ctx.restore();
        }

        /**
         * 绘制转盘border
         */
        drawPanel() {
            const ctx = this.ctx
            ctx.save()
            ctx.drawImage(this.backgroundImage, 0, 0, this.radius * 2, this.radius * 2);
            ctx.restore()
        }

        /**
         * 绘制奖盘
         */
        drawPrizeBlock() {
            const ctx = this.ctx
            const awards = this.mAwards
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
        }

        /**
         * 处理文字换行
         */
        getLineTextList(ctx: CanvasText, text: string, maxLineWidth: number) {
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
        }

        /**
         * 绘制每个奖品的图片
         * 图片必须居于每个扇叶的中线，所以this.imageWidth/2去比上中线的位置，得到一个开始画图片的坐标与中线的角度
         * 扇叶一半的角度 - 上述角度 = 画笔偏移的坐标角度，从这个角度开始画才可以使图片永远平分在中线两侧
         */
        async drawAwardPic() {
            const ctx = this.ctx;
            const width = this.imageWidth;
            const height = this.imageHeight;
            const awards = this.mAwards
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
        }


    }
</script>

<style scoped>

</style>
