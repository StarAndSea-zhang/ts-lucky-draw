<!--
 -->
<template>
    <canvas id="canvas" class="rotate-panel"></canvas>
</template>

<script>
    export default {
        name: "ExtractRotatePanel",
        data() {
            return {
                canvas: '',
                ctx: '',
                startRadian: 0, // 初始角度
                backgroundImage: null,
                tempNames: this.names,
                startTime: 0,
                // 旋转定时器
                rotateTimer: null,
                terminate:false //是否为最终状态 停止动画后,文字不再透明
            }
        },
        props: {
            rate: {type: Number, default: 3600},                   // 转盘速率
            radius: {type: Number, default: 75},                   // 转盘半径
            textFontSize: {type: String, default: '12px'},         // 文字大小
            lineHeight: {type: Number, default: 20},               // 文字行高
            textColor: {type: String, default: '#fff'},            // 文字颜色
            textMargin: {type: Number, default: 16},               // 文字距离边框距离
            textPadding: {type: Number, default: 0},               // 文字补偿宽度
            btnFontSize: {type: String, default: '14px'},          // 按钮文字大小
            btnColor: {type: String, default: '#3297ff'},          // 按钮文件颜色
            btnBorderColor2: {type: String, default: '#fff',},     // 按钮内边框颜色
            btnBgColor: {type: String, default: '#fff'},           // 按钮背景颜色
            btnText: {type: String, default: '抽取中'},            // 按钮内容
            btnRadius: {type: Number, default: 10},                // 按钮半径
            names: {type: Array, default: ['暂无数据','','','']},  // 默认4个
            rotateTime: {type: Number, default: 3000}              //转盘运动的时间 单位ms
        },
        watch: {
            names: {
                immediate: true,
                handler(newVal) {
                    this.tempNames = newVal;
                }
            }
        },
        mounted() {
            this.initCanvas()
        },
        methods: {
            /**
             * 初始化转盘
             */
            async initCanvas() {
                this.canvas = document.querySelector('#canvas')
                this.ctx = this.canvas.getContext('2d')
                this.canvas.width = this.radius * 2
                this.canvas.height = this.radius * 2
                await this.loadLocalBackGround();
                this.render()
                this.play(0)
            },
            async loadLocalBackGround() {
                this.backgroundImage = await this.loadLocalImage('rule-panel-background');
            },
            loadLocalImage(url) {
                return new Promise((resolve, reject) => {
                    const image = new Image();
                    image.src = require('@/common/assets/images/' + url + '.png');
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
             * 处理文字换行
             */
            getLineTextList(ctx, text, maxLineWidth) {
                maxLineWidth += this.textPadding
                let wordList = text.split(''), tempLine = '', lineList = []
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
                ctx.clearRect(0, 0, this.radius * 2, this.radius * 2);
                ctx.drawImage(this.backgroundImage, 0, 0, this.radius * 2, this.radius * 2);
                ctx.restore()

                //随机的名字

                const nameList = [];
                this.tempNames.forEach(item=>{
                    let name = item.length > 4 ? item.slice(0, 4) + '...' : item;
                    name = `● ${name}`
                    nameList.push(name);
                })

                //一象限
                this.drawText(this.radius + Math.cos(Math.PI / 4) * (this.radius - this.textMargin),
                    (this.radius - this.textMargin * 2) * Math.cos(Math.PI / 4),
                    Math.PI / 4,
                    nameList[0], 0, Math.PI / 2)

                // 二象限
                this.drawText(this.radius + (this.radius - this.textMargin) * Math.cos(Math.PI / 4),
                    this.radius + (this.radius - this.textMargin) * Math.cos(Math.PI / 4),
                    3 * Math.PI / 4,
                    nameList[1], Math.PI / 2, Math.PI)

                //三象限
                this.drawText(Math.cos(Math.PI / 4) * this.radius - this.textMargin,
                    this.radius + Math.cos(Math.PI / 4) * (this.radius - this.textMargin)
                    , 5 * Math.PI / 4,
                    nameList[2],
                    Math.PI, 3 * Math.PI / 2)

                //四象限
                this.drawText(Math.cos(Math.PI / 4) * (this.radius - this.textMargin * 2),
                    Math.cos(Math.PI / 4) * (this.radius - this.textMargin * 2), 7 * Math.PI / 4,
                    nameList[3],
                    3 * Math.PI / 2, Math.PI * 2)

            },
            /**
             * 控制随机出现名字的动画和位置
             * @param x 绘制文字的x坐标
             * @param y 绘制文字的y坐标
             * @param rotate 文字旋转的角度
             * @param name 绘制的名字
             * @param startRange 闭区间 [startRange,endRange)
             * @param endRange 开区间
             */
            drawText(x, y, rotate, name, startRange, endRange) {
                // ctx.isPointInPath(x, y)
                //透明的速率
                let opacityRate = 0;
                const ctx = this.ctx
                ctx.save()

                //初始角度α = {2k *PI + α} k∈ 正数， 未做负数相关处理
                let startRadian = (this.startRadian + Math.PI / 2) % (Math.PI * 2);//计算超过 2Π 部分 取余 保持在只转一圈象限的范围内

                //让透明的参数在每个象限定义域都是(0,Π/2) 90度以内
                let opacityX = startRadian % (Math.PI / 2); // 在(0,Π/2)内去计算
                const res = (startRadian >= startRange) && (startRadian < endRange);
                // console.log(startRadian,startRange,endRange,res)

                if (res) {
                    // 根据(0,0) (0,Π/2) 最高点为1 求出 透明度随0-90°变化
                    opacityRate = -16 / (Math.PI * Math.PI) * (opacityX * opacityX) + 8 / Math.PI * opacityX;
                    ctx.globalAlpha = opacityRate
                    //停止旋转时,不透明
                    if(this.terminate)
                        ctx.globalAlpha = 1;
                } else {
                    ctx.globalAlpha = 0
                }
                ctx.fillStyle = this.textColor
                ctx.font = `${this.textFontSize} Arial`
                ctx.translate(
                    x, y
                )
                ctx.rotate(rotate)

                //超过四个字处理文字
                this.getLineTextList(ctx, name, 70).forEach((line, index) => {
                    ctx.fillText(line, -ctx.measureText(line).width / 2, ++index * this.lineHeight)
                })
                ctx.restore()
            },
            drawPrizeBlock() {
                const ctx = this.ctx

                let startRadian = this.startRadian
                let RadianGap = Math.PI * 2 / 8

                let endRadian = startRadian + RadianGap

                //  透明扇叶
                ctx.save()
                ctx.beginPath()
                const gradient = ctx.createLinearGradient(0, 0, 0, 90);
                gradient.addColorStop("0", "rgba(255,255,254, 1)");
                gradient.addColorStop("1", "rgba(255,255,254, 0.1)");

                ctx.fillStyle = gradient;
                ctx.moveTo(this.radius, this.radius);
                ctx.arc(this.radius, this.radius, this.radius, startRadian, startRadian + RadianGap, false);
                ctx.fill();
                ctx.restore();
            },
            windowToCanvas(canvas, e) {
                const canvasPosition = canvas.getBoundingClientRect(), x = e.clientX, y = e.clientY
                return {
                    x: x - canvasPosition.left,
                    y: y - canvasPosition.top
                }
            },
            //开启旋转
            play(e) {
                let dom = document.querySelector('.rotate-panel')
                const canvas = this.canvas
                const ctx = this.ctx
                let loc = e ? this.windowToCanvas(canvas, e) : { // 模拟点击坐标
                    x: dom.offsetWidth / 2,
                    y: dom.offsetHeight / 2
                }
                ctx.beginPath()
                ctx.arc(this.radius, this.radius, 50, 0, Math.PI * 2, false)

                //判断当前左边是否在画的区域内
                if (ctx.isPointInPath(loc.x, loc.y)) {
                    this.startRadian = 0
                    this.startTime = new Date().getTime();
                    this.rotatePanel(6)
                }
            },
            /**
             * 旋转转盘
             * @param distance 旋转弧度
             */
            rotatePanel(distance) {
                distance = Math.random() * 220 + 6
                let changeRadian = (distance - this.startRadian) / this.rate

                //当前弧度
                this.startRadian += changeRadian;
                let endTime = new Date().getTime();

                //退出旋转的条件
                if ((endTime - this.startTime) >= this.rotateTime) {

                    // 停止旋转后,再次渲染一次,保证此时字体不透明
                    this.terminate = true;
                    this.render()
                    return;
                }

                // 如果随机圈数
                // if(this.startRadian > Math.PI*6){
                //     return ;
                // }
                this.render()
                window.requestAnimationFrame(this.rotatePanel.bind(this, distance))
            },
            drawButton() {
                const ctx = this.ctx
                //button背景
                ctx.save()
                ctx.beginPath()
                ctx.fillStyle = this.btnBorderColor2
                ctx.arc(this.radius, this.radius, this.btnRadius - 5, 0, Math.PI * 2, false)
                ctx.fill()
                ctx.restore()

                //绘制button中心文字
                ctx.save()
                ctx.beginPath()
                ctx.fillStyle = this.btnColor
                ctx.font = `${this.btnFontSize} Arial`
                ctx.translate(this.radius, this.radius)
                ctx.fillText(this.btnText, -ctx.measureText(this.btnText).width / 2, 8)
                ctx.restore()
            },
            render() {
                this.drawPanel()
                //画扇叶和随机出现的名字
                this.drawPrizeBlock()
                //中间 - 抽取中按钮
                this.drawButton()
            }
        }
    }
</script>

