<template>
<!--用户登录页面-->
<main>
  <header>
    <div class="head_log">
      <div><img src="https://lmaopay.oss-cn-qingdao.aliyuncs.com/res/yTwoLevelList/20.png" ></div>
    </div>
  </header>

  <center>
    <div class="logcontex">
      <em>你好</em>
      <em>欢迎注册懒猫生活</em>
      <span>海量爆点礼包，全国商户免费领取</span>
    </div>
    <div id="form">
      <div class="input icon-mobile-3">
        <input type="number" placeholder="手机号码" pattern="\d*"  ref="mobile"/>
      </div>
      <div class="input icon-lock-1">
        <input type="number" placeholder="请输入验证码" pattern="\d*" ref="validationCode"/>
        <a @click="validation()">{{codetext}}</a>
      </div>

      <div class="validation clearfix" @chang="btn($event)">
        <input type="checkbox" class="chek" checked onchange="checked=true ;"> <span>《懒猫生活》使用协议</span>
      </div>
      <button class="sub" @click="onSubmit()" >登录</button>
      <span class="remind">未注册账户直接创建新用户账号信息</span>
    </div>
  </center>
</main>
</template>

<script>
  import { http,$_GET} from '../../assets/script/com.js'
  import {AlertModule} from 'vux'
  export default {
    components:{AlertModule},
    data() {
      return {
        user:{},
        show:'receive',
        sendcode:false,
        codetext:'获取验证码',
        wxcode:$_GET['code'],
        userinfo:{},
      };
    },
    created(){

    },
    mounted()
    {
      this.getCode();
    },

    methods:{
      getCode() {
        //如果是微信公众号进入，获取code
        if(typeof(this.wxcode) == "undefined")
        {
          var url= window.location.href;
          window.location.href= "https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx69c96fa2bac491f0&redirect_uri="+encodeURIComponent(url)+"&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect";
        }
        else
        {
          var _slef=this;
          http.post({
              uri: 'Home/Home/wxAuth',
              data: {
                  code: this.wxcode,
              },
              callback:function(res)
              {
                if(res.status==0)
                {
                  AlertModule.show({
                    title:'提示',
                    content:e.info
                  });
                }
                else
                {
                  if(res.body.status == 1)
                    {
                      localStorage.setItem('userinfo',JSON.stringify(res.body.info));

                      if(localStorage.getItem("userinfo"))
                      {
                          if(typeof($_GET['ref']) == "undefined")
                          {
                              window.location.href = 'index.html#/user.html';
                          }
                          else
                          {
                              window.location.href = decodeURIComponent($_GET['ref']);
                          }
                      }
                  }else {
                      _slef.userinfo = res.body.info;
                  }
                }
              }
          })
        }
      },

      /*获取验证码*/
      validation()
      {
        var _self = this;
        http.post({
          uri:'Member/Basic/verification',
          data:{
            mobile: _self.$refs.mobile.value,
            module: 'login'
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
              _self.sendcode = true;
                var time = 60;
              var t = setInterval(()=>
              {
                --time;
                if(time == 0)
                {
                  clearInterval(t);
                  _self.sendcode = false;
                  _self.codetext = '获取验证码';
                }
                else{
                  _self.codetext = time+'后重发';
                }
              },1000)
            }
          }
        })
      },

      /*登录*/
      onSubmit() {
          var _self = this;
          http.post({
            uri:'Member/Registe/login',
            data:{
              mobile: _self.$refs.mobile.value,
              verify: _self.$refs.validationCode.value,
              openid: _self.userinfo.openid,
              unionid: _self.userinfo.unionid,
              avatar: _self.userinfo.avatar,//JSON.stringify(_self.userinfo.avatar)
              nickname: _self.userinfo.nickname,
              module:'login',
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

                localStorage.setItem('userinfo',JSON.stringify(e.body));
                if(localStorage.getItem("userinfo"))
                {
                    if(typeof($_GET['ref']) == "undefined") {
                        window.location.href = 'index.html#/user.html';
                    }
                    else
                    {
                        window.location.href = decodeURIComponent($_GET['ref']);
                    }
                }
                else
                {
                    AlertModule.show({
                        title:'提示',
                        content:'登录失败'
                    });
                }
              }
            }
          })
      },

    }
  }
</script>

<style>
@import "../../assets/style/common.css";
main{overflow-y: auto; -webkit-overflow-scrolling: touch; background-color: white;}

header .head_log {width: 5.44rem;padding: 1rem; text-align: center;}
header .head_log div:first-child {width: 1.27rem; height: 1.27rem; border-radius: .2rem; margin-left: 2.02rem;}
header .head_log div:first-child img {width: 1.58rem; height: 1.64rem; border-radius: .2rem;}
header .head_log div:last-child {font-size: .34rem; font-weight: bold; color: #000000;margin-top: .2rem;}

center #form {width: 6rem; margin-top: .5rem;}
center .logcontex {text-align: left;width: 6rem; margin-top: -.5rem;}
center .logcontex em {font-size: .36rem; font-weight: bold; color: #333333; display: block; display: block; margin-top: .15rem;}
center .logcontex span {font-size: .24rem; font-weight: bold; color: #666666; display: inline-block; margin-top: .15rem;}
center #form .input{ border-bottom: 1px solid #EEEEEE; height: 0.7rem; margin-top: 0.2rem; display: flex; flex-direction:row;  align-items:center;}
center #form .input:before{ width: 0.6rem; height: 0.6rem; font-size: 0.5rem; text-align: left; line-height:0.6rem;}
center #form .input input{ height: 0.6rem; line-height: 0.6rem; font-size:0.42rem; border: 0;}
center #form .input input::-webkit-input-placeholder{ font-size: 0.3rem;}
center #form .input.icon-mobile-3 input{ width:4rem; margin-right: 0.3rem;}
center #form .input.icon-lock-1 input{ width:3.5rem; margin-right: 0.3rem;}
center #form .input a{ height: 0.5rem; width: 1.8rem; flex: 1; color: #F47042; line-height: 0.5rem;  border-radius: 0.08rem; background-color: rgba(243,104,56,.2); }

center #form .validation {width: 100%; margin-top: 0.2rem;text-align: left;}
center #form .validation input {width: .3rem; height: .3rem;border-radius: .3rem;background-color: #EA5504;vertical-align: middle;}
center #form .validation input:nth-of-type(2) {margin-left: 1.9rem;}
center #form .validation span {color: #999999; font-size: .24rem; font-weight: 500;}
center #form .sub{;display: inline-block;width: 6.18rem;border: 0;height: 0.86rem; background: #F36838; border-radius: .5rem; margin-top: 0.4rem; font-size: 0.4rem; color: #FFFFFF; line-height: 0.86rem; text-align: center;}
center .remind {display: inline-block; margin-top: .2rem; font-size: .24rem; font-weight: 500; color: #666666;}
</style>
