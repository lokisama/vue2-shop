<template>
  	<div class="city_container">
        
        <form class="city_form" v-on:submit.prevent>
            <router-link to="/home" slot="changecity" class="change_city_right button_style">上海</router-link>
            <input type="search" name="city" placeholder="请输入要搜索订阅的门店名称" class="city_input input_style" required v-model='inputVaule' @input='postpois'>
            <div class="head_back_left button_style" @click="$router.go(-1)">取消</div>
            <!-- <div>
                <input type="submit" name="submit" class="city_submit input_style" @click='postpois' value="提交">
            </div> -->
        </form>
        <header v-if="historytitle" class="pois_search_history">共36,705,325条</header>
        <ul class="getpois_ul">
            <!-- <li v-for="(item, index) in placelist" @click='nextpage(index, item.geohash)' :key="index"> -->
            <li v-for="(item, index) in placelist" :key="index" @click='active(item)'>
                <a class="add_icon" :class="{'active':item.active === true}">
                    <svg @touchstart="">
                        <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#select"></use>
                    </svg>
                </a>
                <img src="http://images.cangdu.org/15d4a7a730f14.png">
                <div class="pois_detail">
                    <h4 class="pois_name ellipsis">{{item.name}}</h4>
                    <p class="pois_address ellipsis">{{item.address}}</p>
                </div>
            </li>
        </ul>
        <footer v-if="historytitle&&placelist.length" class="clear_all_history" @click="clearAll">清空所有</footer>
        <div class="search_none_place" v-if="placeNone">很抱歉！无搜索结果</div>
        <div class="button_container">
            <a class="button orange">全选</a>
            <a class="button red" @click='nextpage(2, "31.15385,121.30464")'>去订阅</a>
        </div>
    </div>
</template>

<script>
    import {mapState, mapMutations} from 'vuex'
    import {currentcity, searchplace} from 'src/service/getData'
    import {getStore, setStore, removeStore} from 'src/config/mUtils'

    export default {
    	data(){
            return{
                inputVaule:'', // 搜索地址
                cityid:'', // 当前城市id
                cityname:'', // 当前城市名字
                placelist:[], // 搜索城市列表
                placeHistory:[], // 历史搜索记录
                historytitle: true, // 默认显示搜索历史头部，点击搜索后隐藏
                placeNone: false, // 搜索无结果，显示提示信息
            }
        },

        mounted(){
            this.cityid = this.$route.params.cityid;
            //获取当前城市名字
            currentcity(this.cityid).then(res => {
                this.cityname = res.name;
            })
            this.initData();
        },

        components:{
        },

        computed:{

        },

        methods:{
            ...mapMutations([
                'RECORD_ADDRESS','ADD_CART','REDUCE_CART','INIT_BUYCART','CLEAR_CART','RECORD_SHOPDETAIL'
            ]),

            initData(){
                //获取搜索历史记录
                if (getStore('placeHistory')) {
                    this.placelist = JSON.parse(getStore('placeHistory'));
                }else{
                    this.placelist = [];
                }
                this.INIT_BUYCART();
            },
            //发送搜索信息inputVaule
            postpois(){
                //输入值不为空时才发送信息
                if (this.inputVaule) {
                    searchplace(this.cityid, this.inputVaule).then(res => {
                        if(res.name === "ERROR_QUERY_TYPE" || false === res instanceof  Array){
                            return;
                        }
                        this.historytitle = false;
                        this.placelist = res;
                        this.placeNone = res.length? false : true;
                    })
                }
            },
            /**
             * 点击搜索结果进入下一页面时进行判断是否已经有一样的历史记录
             * 如果没有则新增，如果有则不做重复储存，判断完成后进入下一页
             */
            nextpage(index, geohash){
                // console.log(index, geohash);
                this.ADD_CART({shopid: 2, category_id:2, item_id:2, food_id:1, name:'西贝莜面村（上海宝乐汇店）', price:1, specs:""});
                this.$router.push({path:'/confirmOrder', query:{shopId:2,geohash}});
                
            },
            active(item){
                console.log(item);
                item.active = !item.active;
            },
            clearAll(){
                removeStore('placeHistory');
                this.initData();
            }
        }
    }

</script>

<style lang="scss" scoped>
    @import 'src/style/mixin';
    body{
        background-color:#fff!important;
    }
    .button_container{
        position: fixed;
        bottom:0;
        left:0;
        right:0;
        display: flex;
        align-items: flex-end;
        justify-content: center;
        background-color:#fff;
        padding:.4rem 0;
        .button{
            @include wh(90%, auto);
            @include sc(.6rem, #fff);
            text-align: center;
            padding:.4rem 0;
            border-radius: .2rem;
            margin:0 1rem;
            &.orange{
                background-color:#ff6d40;
            }
            &.red{
                background-color:#fc3c3f;
            }
        }
    }
    .city_container{
        padding-top: 2rem;
        padding-bottom: 2rem;
    }
    .change_city{
        right: 0.4rem;
        @include sc(0.6rem, #fff);
        @include ct;
    }
    .city_form{
        position: fixed;
        z-index: 100;
        left: 0;
        top: 0;
        @include wh(100%, 1.95rem);
        background-color: #fff;
        border-top: 1px solid $bc;
        border-bottom: 1px solid $bc;
        display: flex;
        align-items: center;
        justify-content: center;
        @include sc(0.65rem, #333);
        .button_style{

            margin: 0.4rem ;
        }
        .input_style{
            border-radius: 0.1rem;
            margin: 0.4rem 0;
            @include wh(100%, 1.4rem);
        }
        .city_input{
            flex:1;
            border: 1px solid $bc;
            padding: 0 0.3rem;
            @include sc(0.65rem, #333);
        }
        .city_submit{
            background-color: $blue;
            @include sc(0.65rem, #fff);
        }
    }
    .pois_search_history{
        border-top: 1px solid $bc;
        border-bottom: 1px solid $bc;
        padding-left: 0.5rem;
        @include font(0.475rem, 0.8rem);
    }
    .getpois_ul{
        background-color: #fff;
        border-top: 1px solid $bc;
        overflow: hidden;
        li{
            margin: 0 auto;
            border-bottom: 1px solid $bc;
            display: flex;
            flex-direction:row;
            text-align: center;
            padding: .5rem 0;
            img{
                @include wh(70px, 3rem);
            }
            .add_icon{
                padding: 0 .4rem;
                svg{
                    margin-top:1rem;
                    width:.85rem;
                    height:.85rem;
                    fill: #ccc;
                }
                &.active{
                    svg{
                        fill:#fc3c3f;
                    }
                }
            }
            .pois_detail{
                flex:1;
                display: flex;
                flex-direction:column;
                justify-content: center;
                text-align: start;
                .pois_name{
                    margin: 0 auto 0.35rem;
                    width: 90%;
                   @include sc(0.65rem, #333);
                }
                .pois_address{
                    width: 90%;
                    margin: 0 auto 0.55rem;
                    @include sc(0.45rem, #999);
                }
            }
        }
    }
    .search_none_place{
        margin: 0 auto;
        @include font(0.65rem, 1.75rem);
        color: #333;
        background-color: #fff;
        text-indent: 0.5rem;
    }
    .clear_all_history{
        @include sc(0.7rem, #666);
        text-align: center;
        line-height: 2rem;
        background-color: #fff;
    }
    body{
        background-color: #fff!important;
    }
</style>
