<template>
    <div id="detail">
        <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"></detail-nav-bar>
        <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll">                        
            <detail-swiper :top-images="topImages">
            </detail-swiper>
            <detail-base-info :goods="goods"></detail-base-info>
            <detail-shop-info :shop="shop"></detail-shop-info>
            <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"></detail-goods-info>
            <detail-param-info ref="params" :param-info="paramInfo"></detail-param-info>
            <detail-comment-info ref="comment" :comment-info="commentInfo"></detail-comment-info>
            <goods-list ref="recommend" :goods="recommends"></goods-list>
        </scroll>
        <detail-bottom-bar @addCart="addToCart"></detail-bottom-bar>
        <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
        <!-- <toast :message="message" :show="show"></toast> -->
    </div>
</template>
<script>
import DetailNavBar from './childComps/DetailNavBar';
import DetailSwiper from './childComps/DetailSwiper';
import DetailBaseInfo from './childComps/DetailBaseInfo';
import {getDetail, Goods, Shop, GoodsParam, getRecomment} from 'network/detail';
import DetailShopInfo from './childComps/DetailShopInfo';
import Scroll from 'components/common/scroll/Scroll';
import DetailGoodsInfo from './childComps/DetailGoodsInfo';
import DetailParamInfo from './childComps/DetailParamInfo';
import DetailCommentInfo from './childComps/DetailCommentInfo';
import GoodsList from 'components/content/goods/GoodsList';
import {debounce} from 'common/utils';
import {itemListenerMixin, backTopMixin} from 'common/mixin';
import DetailBottomBar from './childComps/DetailBottomBar';
import {BACK_POSITION} from 'common/const';
import {mapActions} from 'vuex';
// import Toast from 'components/common/toast/Toast';
export default {
    name: 'Detail',
    components:{
        DetailNavBar,
        DetailSwiper,
        DetailBaseInfo,
        DetailShopInfo,
        Scroll,
        DetailGoodsInfo,
        DetailParamInfo,
        DetailCommentInfo,
        GoodsList,
        DetailBottomBar,
        // Toast
    },
    mixins: [itemListenerMixin, backTopMixin],
    data() {
        return {
            iid: null,
            topImages:[],
            goods:{},
            shop: {},
            detailInfo:{},
            paramInfo:{},
            commentInfo:{},
            recommends: [],
            themeTopYs: [],
            getThemeTopY: null,
            currentIndex: 0,
            // message:'',
            // show: false
        }
    },
    created() {
        this.iid = this.$route.params.iid;
        getDetail(this.iid).then(res => {
            const data = res.result;
            this.topImages = data.itemInfo.topImages;  
            this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services);
            this.shop = new Shop(data.shopInfo);
            this.detailInfo = data.detailInfo;
            this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule);
            if (data.rate.cRate !== 0) {
                this.commentInfo = data.rate.list[0];
            }
        });
        getRecomment().then(res => {
            this.recommends = res.data.list;         
        });
        this.getThemeTopY = debounce(() => {
            this.themeTopYs = [];
            this.themeTopYs.push(0);
            this.themeTopYs.push(this.$refs.params.$el.offsetTop - 44);
            this.themeTopYs.push(this.$refs.comment.$el.offsetTop - 44);
            this.themeTopYs.push(this.$refs.recommend.$el.offsetTop - 44);
            this.themeTopYs.push(Number.MAX_VALUE);
        }, 100);
    },
    methods: {
        ...mapActions(['addCart']),
        imageLoad() {
            this.$refs.scroll.refresh();
            this.getThemeTopY();
        },
        titleClick(index) {
            this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200);
        },
        contentScroll(position) {
            const positionY = -position.y;
            let length = this.themeTopYs.length;
            for (let i = 0; i < length - 1; i++) {
                if (this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i + 1])) {
                    this.currentIndex = i;
                    this.$refs.nav.currentIndex = this.currentIndex;
                }
            }
            this.isShowBackTop = (-position.y) > BACK_POSITION;
        },
        addToCart() {
            const product = {};
            product.image = this.topImages[0];
            product.title = this.goods.title;
            product.desc = this.goods.desc;
            product.price = this.goods.realPrice;
            product.iid = this.iid;
            this.addCart(product).then(res => {
                this.$toast.show(res, 1500);
            });
        }
    },
    destroyed() {
        this.$bus.$off('itemImgLoad', this.itemImgListener);
    },
};
</script>
<style scoped>
    #detail {
        position: relative;
        z-index: 9;
        background-color: #fff;
        height: 100vh;
    }
    .detail-nav {
        position: relative;
        z-index: 9;
        background-color: #fff;
    }
    .content {
        overflow: hidden;
        height: calc(100% - 44px - 49px);
    }
</style>