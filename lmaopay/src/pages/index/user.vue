<template>
<main>
  <scroller lock-x height="100%" :bounce=false ref="scrollerBottom" :scroll-bottom-offst="200">
    <center id="user">
      <div class="header">
        <img class="avatar" v-lazy="user.avatar" />
        <div class="infoer">
          <div class="nickname"><em>{{user.nickname}}</em><i v-if="user.invite_code !==''">{{user.invite_code}}</i></div>
          <div class="vip"><em>{{user.group_name}}</em><i class="icon-qrcode"></i></div>
        </div>
      </div>
      <div class="tooler">
        <div class="wallet">
          <a href="javascript:;">
            <img v-lazy="'https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/31.png'" />
            <em>我的钱包</em>
          </a>
        </div>
        <div class="amount">
          <!--<a href="balance.html"><i>{{user.amount}}</i><em>余额</em></a>-->
          <a href="javascript:;"><i>{{user.amount}}</i><em>余额</em></a>
          <a href="coupon.html"><i>{{user.coupon}}</i><em>券</em></a>
          <a href="javascript:;"><i>0</i><em>积分</em></a>
        </div>
      </div>
      <div class="aduser">
        <a :href="aduser.url?aduser.url:'javascript:;'"><img v-lazy="aduser.image_url"></a>
      </div>
      <div class="menuer">
        <ul>
          <li><a href="coupon.html"><img v-lazy="'https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/95.png'"><em>我的券</em></a></li>
          <li><a href="order.html"><img v-lazy="'https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/96.png'"><em>订单管理</em></a></li>
          <li><a href="address.html"><img v-lazy="'https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/97.png'"><em>地址管理</em></a></li>
          <li><a href="javascript:;" @click="onReset()"><img v-lazy="'https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/98.png'"><em>清除缓存</em></a></li>
        </ul>
      </div>
    </center>
  </scroller>
  <vfooter></vfooter>
</main>
</template>

<script>
import {Scroller,AlertModule} from 'vux'
import wx from 'weixin-js-sdk';
import { http,$_GET} from '../../assets/script/com.js'
import vfooter from './vfooter.vue';
export default {
  components: {vfooter,Scroller,AlertModule},
	data(){
		return {
      user:{
        avatar:'https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/37.png',
        nickname:'未登录',
        group_name:'普通用户',
        amount:'0.00',
        coupon:0,
      },
      wallet:{

      },
      aduser:{},
		};
	},

	mounted ()
	{
    http.getLogin();

    this.onLoad();
	},
	methods:{
    onLoad()
    {
      let _self = this;
      http.post({
        uri:'Member/Member/myinfo',
        data:{
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
            _self.$set(_self,'aduser',e.body.advert_model);
            if(e.body.userinfo)
            {
              _self.$set(_self,'user',e.body.userinfo);
            }
          }
        }
      })
    },
    onReset()
    {
      localStorage.clear();
      http.getLogin();
    }
  }
}
</script>

<style>
#user{ text-align: left;}
#user .header{ height:1.2rem; padding:0.5rem 0.3rem 1.45rem 0.8rem; display:flex; flex-direction:row; align-items:center; background-color: #FFFFFF; background-image: url(https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/63..png); background-size: 7.5rem; background-position: bottom center; background-repeat: no-repeat;}
#user .header .avatar{ width: 1rem; height:1rem; border-radius:1rem;}
#user .header .infoer{ flex: 1; margin-left: 0.15rem;  color: #FFFFFF;}
#user .header .infoer .nickname{ height: 0.4rem; line-height: 0.4rem;}
#user .header .infoer .nickname em{ font-size: 0.3rem;  margin-right: 0.2rem; padding: 0 0.1rem; max-width:2.5rem; overflow: hidden;}
#user .header .infoer .nickname i:before{ content: '推荐码：';}
#user .header .infoer .vip{ height: 0.4rem; line-height: 0.4rem; margin-top: 0.2rem;}
#user .header .infoer .vip em{ vertical-align: middle; display: inline-block; height: 0.4rem; padding: 0 0.1rem; border-radius: 0.1rem; background: #FF9A41; }
#user .header .infoer .vip i{ vertical-align: middle; width: 0.4rem; height: 0.4rem; display: inline-block; font-size: 0.4rem; margin-left: 0.2rem;}
#user .tooler{ height: 1rem; padding: 0.3rem; background: #FFFFFF; display:flex; flex-direction:row;}
#user .tooler .wallet{ height:1rem; width:1.8rem;}
#user .tooler .wallet a{height:100%; display:flex; flex-direction:column; justify-content:center; align-items:center;}
#user .tooler .wallet a img{ width: 0.5rem; height: 0.5rem;}
#user .tooler .wallet a em{ margin-top: 0.1rem;}
#user .tooler .amount{ flex: 1; height: 1rem; display: flex; flex-direction:row; align-items:center;}
#user .tooler .amount a{ height: 100%; flex: 1; display:flex; flex-direction:column; justify-content:center; align-items:center;}
#user .tooler .amount a i{font-size: 0.3rem; height: 0.5rem; line-height: 0.5rem;}
#user .tooler .amount a em{ margin-top: 0.1rem;}
#user .tooler .amount:before{ content: ''; height:0.5rem; width: 1px; background: #999999;}
#user .aduser{ height: 1.4rem; padding: 0.3rem; background: #FFFFFF; margin-top: 0.3rem;}
#user .aduser a{ display: block; height: 100%;}
#user .aduser a img{ display: block; width: 100%; height: 100%;}
#user .menuer{ background: #FFFFFF; border-top: 1px solid #F2F2F2;}
#user .menuer ul{ padding: 0.15rem;}
#user .menuer ul:after{ content:''; display: block; clear: both;}
#user .menuer ul li{ float: left; height:1.5rem; width: 1.8rem;}
#user .menuer ul li a{display:flex; height:100%; flex-direction:column; justify-content:center; align-items:center;}
#user .menuer ul li a img{ width:0.60rem; height: 0.60rem;}
#user .menuer ul li a em{ margin-top: 0.1rem;}
</style>
