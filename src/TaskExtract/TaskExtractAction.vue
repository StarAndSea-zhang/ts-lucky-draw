<!--
 * @Description: 双随机抽取任务 - 输入任务名称 - 模糊搜索
 * @Author ZhangYu
 * @Date 2020/8/10
 -->
<template>
    <div class="w-extract">
        <!--抽取之前-->
        <div v-if="show" class="w-extract__before">
            <div class="w-extract__banner"></div>
            <!--当前多少规则-->
            <div class="w-extract__search">
                <img src="../../../common/assets/images/rule-icon-search.png" class="w-extract__search--icon"/>
                <a-select
                        show-search
                        :value="value"
                        :placeholder="placeholder"
                        :maxTagCount="10"
                        :dropdownMatchSelectWidth="false"
                        :default-active-first-option="false"
                        :show-arrow="false"
                        :filter-option="false"
                        :not-found-content="null"
                        @search="handleSearch"
                        @change="handleChange"
                >
                    <a-select-option v-for="d in afterSearchData" :key="d.value">
                        {{ d.text }}
                    </a-select-option>
                </a-select>
                <a-icon type="down" class="w-extract__arrow"/>
            </div>
            <div class="button-wrapper">
                <a-button class="button-extract">立即抽取</a-button>
            </div>
        </div>
        <div v-else class="w-extract__after">
            <ExtractAfterBanner></ExtractAfterBanner>
        </div>
    </div>
</template>

<script>
    import jsonp from 'fetch-jsonp';
    import querystring from 'querystring';
    import ExtractAfterBanner from "@/pages/DoublyStochastic/TaskExtract/components/ExtractAfterBanner";

    let timeout;
    let currentValue;

    function fetch(value, callback) {
        if (timeout) {
            clearTimeout(timeout);
            timeout = null;
        }
        currentValue = value;

        function fake() {
            const str = querystring.encode({
                code: 'utf-8',
                q: value,
            });
            jsonp(`https://suggest.taobao.com/sug?${str}`)
                .then(response => response.json())
                .then(d => {
                    if (currentValue === value) {
                        const result = d.result;
                        const data = [];
                        result.forEach(r => {
                            data.push({
                                value: r[0],
                                text: r[0],
                            });
                        });
                        callback(data);
                    }
                });
        }

        timeout = setTimeout(fake, 300);
    }

    export default {
        name: "DebounceSearchInput",
        components: {ExtractAfterBanner},
        data() {
            return {
                afterSearchData: [],
                value: undefined,
                placeholder:'当前共有200条规则',
                show:false
            };
        },
        methods: {
            handleSearch(value) {
                fetch(value, data => (this.afterSearchData = data));
            },
            handleChange(value) {
                console.log(value);
                this.value = value;
                fetch(value, data => (this.afterSearchData = data));
            }
        },
    }
</script>

<style lang="less" scoped>
    @button-width: 80px;
    @gap-width: 20px;
    @banner-width:42%;
    @banner-top-margin:160px;
    @search-icon-width:20px;
    @search-icon-arrow:60px;
    @margin-button-wrapper:20px;

    /*搜索框菜单栏*/
    .dropdown{
        background: #1d52c7;
    }

    .option{
        width: @banner-width;
    }

    .w-extract {
        color: #000;

        .ant-select {
            padding: 7px;
            width: calc(100% - @search-icon-arrow - @search-icon-width - @margin-md);
        }

        /*去掉select的边框*/

        /deep/ .ant-select-selection,
        /deep/ .ant-select-selection--single {
            border: 0;
            box-shadow: 0 0 0 2px rgba(0, 0, 0, 0);

        }

        /deep/ .ant-select-focused .ant-select-selection,
        /deep/ .ant-select-selection:focus,
        /deep/ .ant-select-selection:active {
            border-color: white;
            outline: none;
            box-shadow: 0 0 0 2px rgba(0, 0, 0, 0);
        }

        /deep/ .ant-select-selection:hover {
            border-color: white;
            outline: none;
            box-shadow: none;
        }

        &__search {
            width: @banner-width;
            border: 1px solid #ADADAD;
            border-radius: 7px;
            box-shadow: 1px 3px 4px #BEBEBE;
            margin: 0 auto;
            /*搜索图标*/
            &--icon{
                width: @search-icon-width;
                height: @search-icon-width;
                margin-left: @margin-md;
            }
        }

        &__num {

        }

        /*箭头*/
        &__arrow{
            font-size: 18px;
            color: #999999;
            width: @search-icon-arrow;
            height: 32px;
            line-height: 32px;
            border-left: 1px solid #adadad;
        }

        /*抽取之前*/

        &__before {

        }

        /*点击抽取后*/
        &__after{

        }

        &__banner{
            width: @banner-width;
            min-height: 98px;
            background: url("~@/common/assets/images/rule-before-banner.png") no-repeat ;
            background-size: 100%;
            margin: @banner-top-margin auto @margin-lg auto;
        }
    }

    .button-wrapper {
        width: @button-width+@gap-width;
        height: @button-width+@gap-width;
        border-radius: @button-width+@gap-width;
        background: #e5eaf7;
        text-align: center;
        position: relative;
        margin: @margin-lg*2 auto 0 auto;
    }

    .button-extract {
        width: @button-width;
        height: @button-width;
        border-radius: @button-width;
        background: linear-gradient(to top,#2054c8, #7c99f7);
        color: white;
        position: absolute;
        padding: @padding-xs;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
</style>
