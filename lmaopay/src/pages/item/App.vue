<template>
<main>
  <scroller lock-x height="100%" :bounce=false ref="scrollerBottom">
    <center id="item">
      <div class="header">
        <div class="title"><em>{{item.name}}</em><i>{{item.profit}}</i></div>
        <div class="nameer">{{item.second_name}}</div>
      </div>
      <div class="image">
        <ul>
          <li v-for="row in item.image"><img v-lazy="row"></li>
        </ul>
      </div>
      <div class="price"><i>{{item.amount}}</i></div>
      <div class="list">
        <ul>
          <li v-for="row in gift_detail"><em>{{row.title}}</em><i>{{row.amount}}</i></li>
        </ul>
      </div>
      <div class="explain">
        <div class="item">{{item.validity}}</div>
        <div class="item" v-html="item.use_introduce"></div>
      </div>
    </center>
  </scroller>
  <footer>
    <a :href="'store.html?id='+item.mchid" class="icon-home"><em>店铺</em></a>
    <a href="javascript:;" @click="onShare()" class="icon-share-1"><em>分享</em></a>
    <a href="javascript:;" @click="onBuy()">{{(item.gift_type==1)?(item.flag==0)?'立即购买':'免费领取':'立即购买'}}</a>
  </footer>
  <div class="share ubg"  v-bind:class="{on:share}" @click="onShare('close')"></div>
</main>
</template>

<script>
import {Scroller,AlertModule} from 'vux'
import wx from 'weixin-js-sdk';
import { http,$_GET} from '../../assets/script/com.js'
export default {
  components: {Scroller,AlertModule},
	data(){
		return {
      share:false,
      item:{
        image:[{},{},{},{},{}]
      },
      gift_detail:[]
		};
	},

	mounted ()
	{
    this.onLoad();
	},
	methods:{
    onLoad()
    {
      let _self = this;
      http.post({
        uri:'Merchant/Merchant/gift_detail',
        data:{
          puid:(typeof($_GET['puid'])!='undefined')?$_GET['puid']:'',
          id:$_GET['id'],
          uid:http.getUid()
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
            _self.$set(_self,'gift_detail',e.body.gift_detail);
            _self.$set(_self,'item',e.body.gift);
            // 加载微信分享
            _self.onLoadWxConf();
          }
        }
      })
    },
    onLoadWxConf()
    {
      let _self = this;
      http.post({
        uri:'Home/Home/wxJssdk',
        data:{
          url:window.location.href
        },
        callback:function(e)
        {
          wx.config({
            debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId:e.body.appId, // 必填，公众号的唯一标识
            timestamp:e.body.timestamp , // 必填，生成签名的时间戳
            nonceStr:e.body.nonceStr, // 必填，生成签名的随机串
            signature:e.body.signature,// 必填，签名
            jsApiList: ['updateAppMessageShareData','updateTimelineShareData','hideAllNonBaseMenuItem','showMenuItems'] // 必填，需要使用的JS接口列表
          });

          wx.ready(function ()
          {
            wx.hideAllNonBaseMenuItem();
            wx.showMenuItems({
              menuList: ['menuItem:share:appMessage','menuItem:share:timeline'] // 要显示的菜单项，所有menu项见附录3
            });
            wx.updateAppMessageShareData({
              title:'点击一下看看也不要钱，更何况还能免费领取礼包呢？已有1000万人加入就差你了', // 分享标题
              desc:'懒猫生活服务实体商家共享模式开创者，海量爆点礼包免费领取，最低价值119元。本地生活服务一站式解决', // 分享描述
              link:http.setUrl(window.location.href,{puid:http.getUid()}), // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
              imgUrl:_self.item.image[0], // 分享图标
              success:function()
              {
                // 设置成功
              }
            });
            wx.updateTimelineShareData({
                title:'点击一下看看也不要钱，更何况还能免费领取礼包呢？已有1000万人加入就差你了', // 分享标题
                link:http.setUrl(window.location.href,{puid:http.getUid()}), // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
                imgUrl:_self.item.image[0], // 分享图标
                success: function ()
                {
                  // 设置成功
                }
            });
          });
        }
      })

    },
    onBuy()
    {
      let user = http.getLogin();
      http.post({
        uri:'Member/Coupon/buy_gift',
        data:{
          flag:this.item.flag,
          uid:user.uid,
          gift_id:$_GET['id'],
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
            let ie = e.body;
            if(ie.flag==0)
            {
              WeixinJSBridge.invoke(
                'getBrandWCPayRequest',
                JSON.parse(ie.pay_info),
                function(res)
                {
                  if(typeof(res.err_code)=="undefined")
                  {
                    if(res.err_msg=='get_brand_wcpay_request:ok')
                    {
                      window.location.href = 'coupon.html'
                    }
                    else
                    {
                      AlertModule.show({
                        title:'提示',
                        content:'支付失败'
                      });
                    }
                  }
                  else
                  {
                     AlertModule.show({
                       title:'提示',
                       content:res.err_code
                     });
                  }
                }
              );
            }
            else
            {
              window.location.href = 'coupon.html'
            }
          }
        }
      })
    },
    onShare(close)
    {
      if(typeof(close) !=='undefined' && close =='close')
      {
        this.share = false;
      }
      else
      {
        this.share = true;
      }
    }
  }
}
</script>

<style>
@import "../../assets/style/common.css";
main{ height: 100%; width: 7.5rem; margin: 0 auto; display:flex; flex-direction: column;}
#item{ text-align: left;}
#item .header{ height:0.8rem; padding: 0.2rem 0.3rem 0 0.3rem; background: #FFFFFF;}
#item .header .title{ height: 0.4rem; line-height: 0.4rem;}
#item .header .title em{ font-size:0.32rem; margin-right: 0.1rem;}
#item .header .title i{ color: #DD5410;}
#item .header .title i:before{ content: '分享返利'; margin-right: 0.05rem;}
#item .header .title i:after{ content: '元'; margin-left: 0.05rem;}
#item .header .nameer{ height: 0.4rem; line-height: 0.4rem;}
#item .image{ padding: 0.3rem 0.3rem 0 0.3rem; background: #FFFFFF; height: 2.5rem; overflow: hidden;}
#item .image ul{ height: 2.5rem; width:calc(2.6rem * 6);}
#item .image ul li{ float: left; width: 2.5rem; height: 2.5rem; margin-right: 0.2rem;}
#item .image ul li img{ display: block; width: 2.5rem; height: 2.5rem; border-radius: 0.15rem;}
#item .price{ background: #FFFFFF; height: 0.5rem; line-height: 0.5rem; text-align: right; padding:0.15rem 0.3rem; font-size: 0.42rem;}
#item .price i{ color: #DD5410;}
#item .price i:before{ content: '￥';}

#item .list{ padding:0 0.3rem 0.3rem 0.3rem; background: #FFFFFF;}
#item .list:before{ content: '套餐内容'; clear: both; height: 0.7rem; display: block; font-size: 0.32rem; border-bottom: 1px solid #efefef;}
#item .list ul li{ height: 0.6rem; line-height: 0.6rem; display:flex; flex-direction:row; font-weight: 500;}
#item .list ul li:before{ content: '·'; margin-right: 0.05rem;}
#item .list ul li em{ flex: 1;}
#item .list ul li i{ width: 1.5rem; text-align: right;}
#item .list ul li i:before{ content: '￥';}
#item .explain{ padding: 0.3rem; margin-top: 0.3rem; background: #FFFFFF;}
#item .explain:before{ content: '购买须知'; clear: both; height: 0.7rem; display: block; font-size: 0.32rem;}
#item .explain .item{ margin-bottom: 0.3rem;}
#item .explain .item:before{ clear: both; height: 0.7rem; display: block; color: #999999;}
#item .explain .item:nth-of-type(1):before{ content: '有效期：';}
#item .explain .item:nth-of-type(2):before{ content: '使用规则：';}
footer{ height:1rem; background: #FFFFFF; display:flex; flex-direction:row;}
footer a{ height: 100%; display:flex; flex-direction:column; justify-content:center; align-items:center;}
footer a:before{ font-size: 0.4rem;}
footer a:nth-of-type(1){ width:1.5rem; color: #7b7b7b;}
footer a:nth-of-type(2){ width:1.5rem; color: #DD5410; background: #ffb93e;}
footer a:nth-of-type(3){ flex: 1; color: #FFFFFF; font-size: 0.32rem; background: #DD5410;}
.share{display:none; position: absolute; width: 100%; height: 100%; left:0; top: 0; z-index: 999; }
.share.on{ display:flex; flex-direction:column; align-items:center;}
.share:before{ display: block; height: 5.35rem; width: 5.35rem; content: ''; margin-top: 0.5rem; background-size: 5.24rem; background-image: url(https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/91.png); background-position: center; background-repeat: no-repeat;}
</style>
