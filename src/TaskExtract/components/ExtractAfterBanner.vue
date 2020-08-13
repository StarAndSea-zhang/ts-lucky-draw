<!--
 * @Description:
 * @Author ZhangYu
 * @Date 2020/8/11
 -->
<template>
    <div class="w-banner">
        <div class="w-banner__wrapper">
            <!--抽取中转盘-->
            <extract-rotate-panel :names="extractNameList" :rotate-time="animationTime" class="w-banner__rotate"></extract-rotate-panel>
            <!--右侧面板-->
            <div class="w-banner__right">
                <div class="banner-rule">抽取规则：“生活中的消防知识定期检查”</div>
                <div>
                    <!--进度条展示-->
                    <div class="banner-progress">
                        <span class="banner-progress_number">
                            {{countNum}}%
                        </span>
                    </div>
                    <!--抽取对象随机展示名字的面板-->
                    <extract-target-name class="banner-target" ></extract-target-name>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    import ExtractRotatePanel from "@/pages/DoublyStochastic/TaskExtract/components/ExtractRotatePanel";
    import ExtractTargetName from "@/pages/DoublyStochastic/TaskExtract/components/ExtractTargetName";

    const FINAL_PROGRESS = 100;
    export default {
        name: "ExtractAfterBanner",
        components: {ExtractTargetName, ExtractRotatePanel},
        data() {
            return {
                extractNameList:[ '1号','2号','3号','4号'],
                countNum:0,
                progressTimer:null,
                FINAL_PROGRESS,
                animationTime:6000 //单位：毫秒数 ms
            }
        },
        computed:{
        },
        methods:{
            startProgress(){
                this.progressTimer =  setInterval(()=>{
                    this.countNum++;
                    if(this.countNum>=this.FINAL_PROGRESS){
                        clearInterval(this.progressTimer)
                    }
                },this.animationTime/100); //每0.06s跳动一次
                return this.countNum;
            }
        },
        created() {
            this.startProgress();
        }
    }
</script>

<style lang="less" scoped>
    @wrapper-height: 150px;
    .w-banner {
        /*背景*/

        &__wrapper {
            width: 80%;
            margin: 0 auto;
            background: #f8f8f8;
            height: @wrapper-height;
        }

        /*抽取转盘*/

        &__rotate {
            margin: 0 @margin-lg;
            float: left;
        }

        &__right {
            .banner-rule {
                font-size: 18px;
                font-weight: 600;
                font-family: SimHei;
                padding: @padding-xs 0;
                color: @color-font-black;
            }

            /*进度显示*/

            .banner-progress {
                float: left;
                width: 111px;
                height: 73px;
                background: url("~@/common/assets/images/rule-panel-progress.png") no-repeat;
                background-size: 100%;
                position: relative;

                &_number {
                    font-size: 30px;
                    font-weight: bold;
                    color: #3297ff;
                    position: absolute;
                    left: 50%;
                    transform: translateX(-50%);
                    bottom: 4px;
                }
            }
            /*抽取对象随机展示名字的面板*/
            .banner-target{
                overflow: hidden;
            }
        }
    }
</style>
