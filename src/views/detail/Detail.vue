<template>
  <div  id="detail">
    <detail-nav-bar class="detail-nav" @titleClick='titleClick' ref="nav"/>
    <scroll class="content" ref="scroll" 
    :probeType="3" @scroll="contentScroll">
      <detail-swiper :top-images="topImages" />
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad='imageLoad'/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :commentInfo="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>

    <detail-bottom-bar @addCart="addToCart"/>

    <!-- <toast :message="message" :show="show" /> -->
  </div>
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar'
import DetailSwiper from './childComps/DetailSwiper'

import {getDetail,Goods,Shop,GoodsParam,getRecommend} from 'network/detail'
import DetailBaseInfo from './childComps/DetailBaseInfo.vue'
import DetailShopInfo from './childComps/DetailShopInfo.vue'
import Scroll from '../../components/common/scroll/Scroll.vue'
import DetailGoodsInfo from './childComps/DetailGoodsInfo.vue'
import DetailParamInfo from './childComps/DetailParamInfo.vue'
import DetailCommentInfo from './childComps/DetailCommentInfo.vue'
import GoodsList from '../../components/content/goods/GoodsList.vue'
import {itemListenerMixin} from 'common/mixin'
import DetailBottomBar from './childComps/DetailBottomBar'
// import Toast from '../../components/common/toast/Toast.vue'

export default {
  name:"Detail",
  components: { 
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    GoodsList,
    DetailBottomBar
    // Toast
    },
  mixins:[itemListenerMixin],
  data() {
    return {
      iid: null,
      topImages: [],
      goods:{},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      itemImgListener:null,
      themeTopYs:[],
      // message:'',
      // show:false
    }
  },
  created() {
    // 1.保存传入的iid
    this.iid = this.$route.params.iid

    // 2。根据iid请求详细数据
    getDetail(this.iid).then(res => {
      // 1.获取顶部的图片轮播图数据
      const data = res.result
      this.topImages = data.itemInfo.topImages
      
      // 2.获取商品信息
      this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)

      // 3.创建店铺信息对象
      this.shop = new Shop(data.shopInfo)

      // 4.保存商品的详情数据
      this.detailInfo = data.detailInfo

      // 5.获取参数的信息
      this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)

      // 6.取出评论的信息
      if(data.rate.cRate !== 0){
        this.commentInfo = data.rate.list[0]
      }
    })

     // 3.请求推荐数据
    getRecommend().then(res=> {
      this.recommends = res.data.list
    })
  },
  mounted() {

    },
  updated() {
    

  },

  destroyed() {
    this.$bus.$off("itemImageLoad",this.itemImgListener)
  },

  methods: {
    imageLoad() {
      this.refresh()
      this.themeTopYs = []
      this.themeTopYs.push(0)
      this.themeTopYs.push(this.$refs.params.$el.offsetTop)
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
      this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
      this.themeTopYs.push(Number.MAX_VALUE)
    },
    titleClick(index) {
      // console.log(index);
      this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],100)
    },
    contentScroll(position) {
      // 1.获取y值
      let positionY = -position.y
      let length = this.themeTopYs.length;
	      for (let i = 0; i < length-1; i++) {
          if(positionY>=this.themeTopYs[i]&&positionY<this.themeTopYs[i+1]){
            this.$refs.nav.currentIndex= i
          }
      }
    },
    addToCart(){
      // 1.获取购物车需要展示的数据
      const product = {}
      product.image = this.topImages[0]
      product.title = this.goods.title
      product.desc = this.goods.desc
      product.price = this.goods.realPrice
      product.iid = this.iid

      // 2.将商品添加到购物车
      // this.$store.cartList.push(product)
      this.$store.dispatch('addCart',product).then(res=>{
        // this.show = true
        // this.message = res

        // setTimeout(() => {
        //   this.show = false
        // }, 1500);
        this.$toast.show(res,2000)
        console.log(this.$toast);
      })
    }
  }
}
</script>

<style scoped>
  #detail {
    position: relative;
    z-index:9;
    background-color: #fff;
    height: 100vh;
  }

  .detail-nav {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 9;
    background-color: #fff;
  }

  .content {
    height: calc(100% - 44px);
    overflow: hidden;
  }

  /* .content {
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  } */
</style>