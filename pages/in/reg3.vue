<template>
	<view class="zai-box">
		<image src="/static/theme/default/register.png" mode='aspectFit' class="zai-logo"></image>
		<view class="zai-title">密聊</view>
		<view class="zai-form">
			<input class="zai-input"  v-model="form.username" v-if="mobileCode" placeholder="请输入手机号" />
			<input class="zai-input"  v-model="form.username" v-else  placeholder="请输入用户名" />
			 <view class="form-code" v-if="mobileCode">
				 <input class="code-input"  v-model="form.sms_code"  placeholder="请输入验证码" />
				 <button class="code" @tap="getCode" v-if="showText"> 获取验证码</button>
				 <button class="code" v-else>{{second}}重新发送</button>
			 </view>

			<!-- <input class="zai-input" placeholder-class password placeholder="请输入密码"/> -->
			<input class="zai-input"  v-model="form.password"  type="password" placeholder="请输入密码"/>
			<button :class="['landing',checkIn ? 'landing_true' : 'landing_false']"
              :disabled="checkIn ? false : true" class="zai-btn" @tap="subReg">立即注册</button>
			<navigator url="./login" open-type='navigateBack' hover-class="none" class="zai-label">已有账号，点此去登录.</navigator>
		</view>
	</view>
</template>

<script>
	import _get from "../../common/_get";
export default {
  data () {
    return {
        showPassword: true,
		showText:true,
		second:60,
		mobileCode:1,
      form: {
        username: '',
        password: '',
		client_id:'',
		  sms_code:''
      }
    }
  },
  onLoad () {

  },
  onShow(){
  	console.log(this.mobileCode);
  	_get.getSysConfig({},function (res) {
		console.log(res)
	})
	let _this = this;
	// #ifdef APP-PLUS
	 plus.push.getClientInfoAsync(function(info){
	     _this.form.client_id = info.clientid;
	},function(e){
		 console.log('Failed');
		 console.log(JSON.stringify(e));
	});  
	// #endif
  },
  computed: {
    checkIn () {
      return this.form.password != '' && this.form.username != '' && this.form.password.length > 5 && this.form.username.length > 5;
    }
  },
  methods: {
	  getCode(){//倒计时
	  	  //获取验证码
		  this.showText = false;
		  var interval = setInterval(() => {
			  let times = --this.second;
			  times = times < 0 ? 0 : times;
			  this.second = times<10?'0'+times:times //小于10秒补 0
		  }, 1000)
		  setTimeout(() => {
			  clearInterval(interval)
			  this.second=60
			  this.showText = true
		  }, 60000)
	  },
	  changePassword () {
      this.showPassword = !this.showPassword;
    },
    delInputUsernameText () {
      this.form.username = ''
    },
    delInputPasswordText () {
      this.form.password = ''
    },
    subReg () {
      let _this = this;
      if (!_this.checkIn) {
        return;
      }
      if (!(/^\w{1,20}$/.test(this.form.username))) {
        uni.showModal({
          content: '密聊号只能包括下划线、数字、字母,并且不能超过20个',
        });
        return;
      }

      if (!(/^\w{1,20}$/.test(this.form.password))) {
        uni.showModal({
          content: '密码只能包括下划线、数字、字母,长度6-20位',
        });
        return;
      }
      uni.showLoading({title:'注册中'})
      _this.$httpSend({
        path: '/im/in/reg',
        data: _this.form,
        success: (data) => {
          uni.setStorage({
            key: 'token',
            data: data.token,
            fail: () => {
              uni.showModal({
                content: '本地存储数据不可用!',
              });
            },
            success () {
              _this.$socketSend({
                action: 'checkToken',
                data: data.token,
              },function () {
                uni.hideLoading()
                uni.reLaunch({
                  url: '../chat/index',
                });
              })

            },
          });
        }
      });
    },
    go_forget () {
      uni.navigateTo({
        url: '../../pages/in/forget'
      })
    },
    go_register () {
      uni.navigateTo({
        url: '../../pages/in/reg'
      })
    }

  }
}
</script>

<style>
	page{
		background-color: #FFFFFF;
	}
	.zai-box{
		padding: 0 100upx;
		position: relative;
	}
	.zai-logo{
		width: 100%;
		width: 100%;
		height: 310upx;
	}
	.zai-title{
		position: absolute;
		top: 0;
		line-height: 360upx;
		font-size: 68upx;
		color: #fff;
		text-align: center;
		width: 100%;
		margin-left: -100upx;
	}
	.zai-form{
		margin-top: 300upx;
	}
	.form-code{
		margin-top: 30upx;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100%;
	}
	.form-code .code{
		height: 100%;
		border-radius: 50px;
		text-align: center;
		font-size: 14px;
		background: #ff65a3;
		color: #ffffff;
	}
	.zai-input{
		background: #e2f5fc;
		margin-top: 30upx;
		border-radius: 100upx;
		padding: 20upx 40upx;
		font-size: 36upx;
	}

	.code-input{
		background: #e2f5fc;
		border-radius: 100upx;
		padding: 20upx 40upx;
		font-size: 36upx;
		width: 40%;
	}
	.input-placeholder, .zai-input{
		color: #94afce;
	}
	.zai-label{
		padding: 60upx 0;
		text-align: center;
		font-size: 30upx;
		color: #a7b6d0;
	}
	.zai-btn{
		background: #ff65a3;
		color: #fff;
		border: 0;
		border-radius: 100upx;
		font-size: 36upx;
		margin-top: 60upx;
	}
	.zai-btn:after{
		border: 0;
	}
	/*按钮点击效果*/
	.zai-btn.button-hover{
		transform: translate(1upx, 1upx);
	}
</style>
