<template>
<!--券-->
<scroller lock-x height="100%" :bounce=false ref="scrollerBottom">
  <center>
    <div v-if="list.length > 0" class="vou_list" v-for="item in list">
      <div class="fonw">
        <img v-lazy="item.gift_info.image">
        <div class="title_fons">
          <em>{{item.gift_info.name}}</em><br/>
          <span id="y">有效期{{item.time_lon}}</span>
        </div>
        <div class="dot">
          <span class="dotone"></span>
          <span class="dottwo"></span>
          <span class="dotthre"></span>
        </div>
        <button @click="onVoucherDetai(item.id)">立即使用</button>
      </div>
    </div>
    <div v-else class="empty"><a href="index.html#/index.html">^_^您还没有券去领取</a></div>
  </center>
</scroller>
</template>

<script>
import { http,$_GET} from '../../assets/script/com.js'
import {Scroller,AlertModule} from 'vux'

export default {
  components: {Scroller,AlertModule},
  data() {
      return {
        user:{},
        list:[{}],
      };
  },
  created(){
    this.user = http.getLogin();
  },
  mounted(){
      this.onLoad();
  },
  methods: {
    onLoad()
    {
      var _self = this;

      http.post({
          uri:'Member/Coupon/gift_list',
          data:{
              uid:this.user.uid,
          },
          callback:function(e)
          {
            if(e.status==0)
            {
              AlertModule.show({
                title:'提示',
                content:e.info
              });
            }
            else
            {
              _self.$set(_self,'list',e.body.gift_list);
            }
          }
      })
    },

    //券详情页
    onVoucherDetai(id){
      window.location.href = 'useCoupon.html?id='+id
    },


  }
}
</script>

<style>
@import "../../assets/style/common.css";
center{ padding: 0.3rem;}
center .vou_list {width: 6.9rem; height: 2.31rem; background-color: white; border-radius: .16rem; margin-bottom: .2rem; text-align: left; position: relative;}
center .vou_list .fonw img {width: 1.19rem; height: 1.19rem; margin-top: .2rem; display: inline-block; margin-left: .4rem; vertical-align: middle;}
center .vou_list .fonw .title_fons {display: inline-block; margin-left: .2rem; position: absolute; top: .35rem;}
center .vou_list .fonw .title_fons em {font-size: .3rem; font-weight: bold; color: #323333;}
center .vou_list .fonw .title_fons #y {font-size: .24rem; font-weight: 500; color: #999999; display: inline-block; margin-top: .15rem;}
center .vou_list .fonw .dot {width: 100%; height: .5rem; position: relative; margin-top: -.05rem;}
center .dot .dotone {margin-left: -.2rem;}
center .dot .dotone, .dotthre{width: .4rem; height: .4rem; background-color: cyan; border-radius: 50%; display: inline-block; background-color: #F2F2F2;}
center .dot .dotthre {float: right; margin-right: -.2rem;}
center .dot .dottwo {width: 6.45rem; height: .01rem; margin-left: .02rem;border-bottom: .01rem dotted #D8D8D8;display: inline-block; position: absolute; top: .2rem;}
center .vou_list .fonw button { outline:none; width: 2.1rem; height: .5rem; margin-top: -.1rem; background-color: #EA5504; color: white; font-size: .24rem; font-weight: 500; border-radius: .4rem; border: 0; float: right; margin-right: .3rem;}
center .empty{ height: 1rem; display:flex;  flex-direction:column;justify-content:center; align-items:center;}
center .empty a{ color: #979797; padding: 0.3rem;}
</style>
