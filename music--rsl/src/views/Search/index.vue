<template>
    <div>
        <van-search v-model="value" placeholder="请输入搜索关键词" shape="round" @input="inputFn" />
        <div class="search_wrap" v-if="resultList.length===0">
            <p class="hot_title">热门搜索</p>
            <div class="hot_name_wrap">
                <span class="hot_item" v-for="(obj,index) 
                 in hotarr" :key="index" @click="fn(obj.first)">
                    {{obj.first}}</span>
            </div>
        </div>
    <div class="search_wrap" v-else>
        <p class="hot_title">最佳匹配</p>
        <van-cell :title="obj.name" :label="obj.ar[0].name" center v-for="obj in resultList" :key="obj.id" >
            <template #right-icon>
                <van-icon name="play-circle-o" size="0.6rem" @touchstart="playFn"/>
               
            </template>
        </van-cell>
       
    </div>
    </div>
</template>

<script>
import { searchAPI, searchResultListAPI } from '@/api'

export default {
    data() {
        return {
            value: '',
            hotarr: [],
            resultList: [],
            timer:null//防抖
            
        }
    },
    async created() {
        const res = await searchAPI()
        this.hotarr = res.data.result.hots
    },
    methods: {
     
        async getListFn() {
            return await searchResultListAPI({
                keywords: this.value,
                limit: 20,
            }); // 把搜索结果return出去
            // (难点):
            // async修饰的函数 -> 默认返回一个全新Promise对象
            // 这个Promise对象的结果就是async函数内return的值
            // 拿到getListFn的返回值用await提取结果
        },
        playFn() {
            this.$router.push({
                path: '/play',
                query: {
                    id: this.id // 歌曲id, 通过路由跳转传递过去
                }
            })
        },
        async fn(val) {
            this.value = val;

            const res = await this.getListFn()

            this.resultList = res.data.result.songs;
            console.log(res);
        },
        async inputFn(){//输入框值改变调用
            if (this.timer) clearTimeout(this.timer)
            this.timer = setTimeout(async () => {
            if (this.value.length === 0) {
                // 搜索关键词如果没有, 就把搜索结果清空阻止网络请求发送(提前return)
                this.resultList = [];
                return;
            }
            const res = await this.getListFn()
                this.resultList = res.data.result.songs;
            }, 900)
        }
    }
}
</script>

<style scoped>
.search_wrap {
    padding: 0.266667rem;
}

/*热门搜索文字标题样式 */
.hot_title {
    font-size: 0.32rem;
    color: #666;
}

/* 热搜词_容器 */
.hot_name_wrap {
    margin: 0.266667rem 0;
}

/* 热搜词_样式 */
.hot_item {
    display: inline-block;
    height: 0.853333rem;
    margin-right: 0.213333rem;
    margin-bottom: 0.213333rem;
    padding: 0 0.373333rem;
    font-size: 0.373333rem;
    line-height: 0.853333rem;
    color: #333;
    border-color: #d3d4da;
    border-radius: 0.853333rem;
    border: 1px solid #d3d4da;
}
</style>
