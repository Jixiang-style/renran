<template>
  <div class="sign">
    <div class="logo">
      <router-link to="/"><img src="/static/image/nav-logo.png" alt="Logo"></router-link>
    </div>
    <div class="main">
      <h4 class="title">
        <div class="normal-title">
          <router-link to="/login">登录</router-link>
          <b>·</b>
          <router-link id="js-sign-up-btn" class="active" to="/register">注册</router-link>
        </div>
      </h4>

      <div class="js-sign-up-container">
        <form class="new_user" id="new_user" action="" accept-charset="UTF-8" method="post">
          <div class="input-prepend restyle">
            <input placeholder="你的昵称" type="text" value="" v-model="nickname" id="user_nickname">
            <i class="iconfont ic-user"></i>
          </div>
          <div class="input-prepend restyle no-radius js-normal">
            <input placeholder="手机号" type="tel" @blur="check_mobile" v-model="mobile" id="user_mobile_number">
            <i class="iconfont ic-phonenumber"></i>
          </div>
          <div class="input-prepend restyle no-radius security-up-code js-security-number" v-if="is_show_sms_code">
            <input type="text" v-model="sms_code" id="sms_code" placeholder="手机验证码">
            <i class="iconfont ic-verify"></i>
            <a tabindex="-1" class="btn-up-resend js-send-code-button" :class="{disable:send_able}"
               href="javascript:void(0);" id="send_code" @click="send_sms">{{ sms_code_text }}</a>
          </div>
          <input type="hidden" name="security_number" id="security_number">
          <div class="input-prepend">
            <input placeholder="设置密码" type="password" v-model="password" id="user_password">
            <i class="iconfont ic-password"></i>
          </div>
          <input type="submit" name="commit" @click.prevent="registerHandler" value="注册" class="sign-up-button"
                 id="sign_up_btn" data-disable-with="注册">
          <p class="sign-up-msg">点击 “注册” 即表示您同意并愿意遵守荏苒<br> <a target="_blank" href="">用户协议</a> 和 <a target="_blank"
                                                                                                    href="">隐私政策</a> 。
          </p>
        </form>
        <!-- 更多注册方式 -->
        <div class="more-sign">
          <h6>社交帐号直接注册</h6>
          <ul>
            <li><a id="weixin" class="weixin" target="_blank" href=""><i class="iconfont ic-wechat"></i></a></li>
            <li><a id="qq" class="qq" target="_blank" href=""><i class="iconfont ic-qq_connect"></i></a></li>
          </ul>

        </div>
      </div>

    </div>
  </div>
</template>

<script>
export default {
  name: "Register",
  data() {
    return {
      nickname: "",
      mobile: "",
      sms_code: "",
      password: "",
      sms_code_text: "发送验证码",
      is_show_sms_code: false,
      send_able: false,
    }
  },
  watch: {
    mobile() {
      // js里面的正则,类似于python re.match
      if (/^1[3-9]\d{9}$/.test(this.mobile)) {
        this.is_show_sms_code = true;
        this.send_able = false;
      } else {
        this.send_able = true;
        this.is_show_sms_code = true;
      }
    }
  },
  methods: {
    check_mobile() {
      // 验证手机号是否唯一
      if (this.is_show_sms_code) {
        // 发送ajax到服务端验证手机号是否可用
        // this.$axios.get(`${this.$settings.Host}/users/mobile/${this.mobile}/`).then(response => {
        this.$axios.post(`${this.$settings.Host}/api/send_code/`).then(response => {
          this.send_able = false;
        }).catch(error => {
          this.$message.error(error.response.data.err_msg);
          this.send_able = true;
        });
      }
    },
    registerHandler() {
      // 注册处理
      if (this.nickname.length < 1 || this.mobile.length < 1 || this.sms_code.length < 1 || this.password.length < 1) {
        this.$message.error("对不起,表单信息不能为空!");
        return false;
      }
      // 发送ajax请求
      this.$axios.post(`${this.$settings.Host}/api/register`, {
        nickname: this.nickname,
        mobile: this.mobile,
        password: this.password,
        sms_code: this.sms_code,
      }).then(response => {
        // 注册成功,则保存用户登录状态
        sessionStorage.user_token = response.data.token;
        sessionStorage.user_name = response.data.username;
        sessionStorage.user_id = response.data.id;
        sessionStorage.user_nickname = response.data.nickname;
        sessionStorage.user_avatar = response.data.avatar;
        localStorage.removeItem("user_token");
        localStorage.removeItem("user_name");
        localStorage.removeItem("user_id");
        localStorage.removeItem("user_nickname");
        localStorage.removeItem("user_avatar");

        // 登录跳转
        console.log("收到的信息",response.data)
        console.log("收到的昵称",response.data.nickname)
        this.$confirm(`${response.data.nickname},欢迎来到荏苒~`, '注册成功', {
          confirmButtonText: '去个人中心',
          cancelButtonText: '返回上一页',
          type: 'success'
        }).then(() => {
          // 跳转到个人中心
          this.$router.push("/user");
        }).catch(() => {
          // 跳转到上一页
          this.$router.back(); // this.$router.go(-1);
        });

      }).catch(error => {
        console.log(error.response);
        this.$message.error("会员注册失败!");
      });
    },
    send_sms() {
      // 发送短信
      if (!/^1[3-9]\d{9}$/.test(this.mobile)) {
        return false;
      }

      if (this.sms_code_text != "发送验证码") {
        return false;
      }

      // this.$axios.get(`${this.$settings.Host}/users/sms/${this.mobile}/`).then(response => {
      this.$axios.post(`${this.$settings.Host}/api/send_code`, {phoneNumber: this.mobile}).then(response => {
        this.$message(response.data.message);
        // 发送短信冷却倒计时
        let timer = 60;
        let t = setInterval(() => {
          if (--timer < 1) {
            this.sms_code_text = "发送验证码";
            clearInterval(t);
          } else {
            this.sms_code_text = `${timer}秒后重新点击`;
            // this.send_able = true;
          }
        }, 1000);
      }).catch(error => {
        this.$message(error.response.data.message);
      });
    }
  }
}
</script>

<style scoped>
input {
  outline: none;
}

*, :after, :before {
  box-sizing: border-box;
}

.sign {
  height: 100%;
  min-height: 750px;
  text-align: center;
  font-size: 14px;
  background-color: #f1f1f1
}

.sign:before {
  content: "";
  display: inline-block;
  height: 85%;
  vertical-align: middle
}

.sign .disable, .sign .disable-gray {
  opacity: .5;
  pointer-events: none
}

.sign .disable-gray {
  background-color: #969696
}

.sign .tooltip-error {
  font-size: 14px;
  line-height: 25px;
  white-space: nowrap;
  background: none
}

.sign .tooltip-error .tooltip-inner {
  max-width: 280px;
  color: #333;
  border: 1px solid #ea6f5a;
  background-color: #fff
}

.sign .tooltip-error .tooltip-inner i {
  position: static;
  margin-right: 5px;
  font-size: 20px;
  color: #ea6f5a;
  vertical-align: middle
}

.sign .tooltip-error .tooltip-inner span {
  vertical-align: middle;
  display: inline-block;
  white-space: normal;
  max-width: 230px
}

.sign .tooltip-error.right .tooltip-arrow-border {
  border-right-color: #ea6f5a
}

.sign .tooltip-error.right .tooltip-arrow-bg {
  left: 2px;
  border-right-color: #fff
}

.sign .slide-error {
  position: relative;
  padding: 10px 0;
  border: 1px solid #c8c8c8;
  border-radius: 4px
}

.sign .slide-error i {
  position: static !important;
  margin-right: 10px;
  color: #ea6f5a !important;
  vertical-align: middle
}

.sign .slide-error span {
  font-size: 15px;
  vertical-align: middle
}

.sign .slide-error div {
  margin-top: 10px;
  font-size: 13px
}

.sign .slide-error a {
  color: #3194d0
}

.sign .js-sign-up-forbidden {
  color: #999;
  padding: 80px 0 100px
}

.sign .js-sign-up-container .slide-error {
  border-bottom: none;
  border-radius: 0
}

.sign .logo {
  position: absolute;
  top: 56px;
  margin-left: 50px
}

.sign .logo img {
  width: 100px
}

.sign .main {
  width: 400px;
  margin: 60px auto 0;
  padding: 50px 50px 30px;
  background-color: #fff;
  border-radius: 4px;
  box-shadow: 0 0 8px rgba(0, 0, 0, .1);
  vertical-align: middle;
  display: inline-block
}

.sign .reset-title, .sign .title {
  margin: 0 auto 50px;
  padding: 10px;
  font-weight: 400;
  color: #969696
}

.sign .reset-title a, .sign .title a {
  padding: 10px;
  color: #969696
}

.sign .reset-title a:hover, .sign .title a:hover {
  border-bottom: 2px solid #ea6f5a
}

.sign .reset-title .active, .sign .title .active {
  font-weight: 700;
  color: #ea6f5a;
  border-bottom: 2px solid #ea6f5a
}

.sign .reset-title b, .sign .title b {
  padding: 10px
}

.sign .reset-title {
  color: #333;
  font-weight: 700
}

.sign form {
  margin-bottom: 30px
}

.sign form .input-prepend {
  position: relative;
  width: 100%
}

.sign form .input-prepend input {
  width: 100%;
  height: 50px;
  margin-bottom: 0;
  padding: 4px 12px 4px 35px;
  border: 1px solid #c8c8c8;
  border-radius: 0 0 4px 4px;
  background-color: hsla(0, 0%, 71%, .1);
  vertical-align: middle
}

.sign form .input-prepend i {
  position: absolute;
  top: 14px;
  left: 10px;
  font-size: 18px;
  color: #969696
}

.sign form .input-prepend span {
  color: #333
}

.sign form .input-prepend .ic-show {
  top: 18px;
  left: auto;
  right: 8px;
  font-size: 12px
}

.sign form .geetest-placeholder {
  height: 44px;
  border-radius: 4px;
  background-color: hsla(0, 0%, 71%, .1);
  text-align: center;
  line-height: 44px;
  font-size: 14px;
  color: #999
}

.sign form .restyle {
  margin-bottom: 0
}

.sign form .restyle input {
  border-bottom: none;
  border-radius: 4px 4px 0 0
}

.sign form .no-radius input {
  border-radius: 0
}

.sign form .slide-security-placeholder {
  height: 32px;
  background-color: hsla(0, 0%, 71%, .1);
  border-radius: 4px
}

.sign form .slide-security-placeholder p {
  padding-top: 7px;
  color: #999;
  margin-right: -7px
}

.sign .overseas-btn {
  font-size: 14px;
  color: #999
}

.sign .overseas-btn:hover {
  color: #2f2f2f
}

.sign .remember-btn {
  float: left;
  margin: 15px 0
}

.sign .remember-btn span {
  margin-left: 5px;
  font-size: 15px;
  color: #969696;
  vertical-align: middle
}

.sign .forget-btn {
  float: right;
  position: relative;
  margin: 15px 0;
  font-size: 14px
}

.sign .forget-btn a {
  color: #999
}

.sign .forget-btn a:hover {
  color: #333
}

.sign .forget-btn .dropdown-menu {
  top: 20px;
  left: auto;
  right: 0;
  border-radius: 4px
}

.sign .forget-btn .dropdown-menu a {
  padding: 10px 20px;
  color: #333
}

.sign #sign-in-loading {
  position: relative;
  width: 20px;
  height: 20px;
  vertical-align: middle;
  margin-top: -4px;
  margin-right: 2px;
  display: none
}

.sign #sign-in-loading:after {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: transparent
}

.sign #sign-in-loading:before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  margin: -10px 0 0 -10px;
  border-radius: 10px;
  border: 2px solid #fff;
  border-bottom-color: transparent;
  vertical-align: middle;
  -webkit-animation: rolling .8s infinite linear;
  animation: rolling .8s infinite linear;
  z-index: 1
}

.sign .sign-in-button, .sign .sign-up-button {
  margin-top: 20px;
  width: 100%;
  padding: 9px 18px;
  font-size: 18px;
  border: none;
  border-radius: 25px;
  color: #fff;
  background: #42c02e;
  cursor: pointer;
  outline: none;
  display: block;
  clear: both
}

.sign .sign-in-button:hover, .sign .sign-up-button:hover {
  background: #3db922
}

.sign .sign-in-button {
  background: #3194d0
}

.sign .sign-in-button:hover {
  background: #187cb7
}

.sign .btn-in-resend, .sign .btn-up-resend {
  position: absolute;
  top: 7px;
  right: 7px;
  width: 100px;
  height: 36px;
  font-size: 13px;
  color: #fff;
  background-color: #42c02e;
  border-radius: 20px;
  line-height: 36px
}

.sign .btn-in-resend {
  background-color: #3194d0
}

.sign .sign-up-msg {
  margin: 10px 0;
  padding: 0;
  text-align: center;
  font-size: 12px;
  line-height: 20px;
  color: #969696
}

.sign .sign-up-msg a, .sign .sign-up-msg a:hover {
  color: #3194d0
}

.sign .overseas input {
  padding-left: 110px !important
}

.sign .overseas .overseas-number {
  position: absolute;
  top: 0;
  left: 0;
  width: 100px;
  height: 50px;
  font-size: 18px;
  color: #969696;
  border-right: 1px solid #c8c8c8
}

.sign .overseas .overseas-number span {
  margin-top: 17px;
  padding-left: 35px;
  text-align: left;
  font-size: 14px;
  display: block
}

.sign .overseas .dropdown-menu {
  width: 100%;
  max-height: 285px;
  font-size: 14px;
  border-radius: 0 0 4px 4px;
  overflow-y: auto
}

.sign .overseas .dropdown-menu li .nation-code {
  width: 65px;
  display: inline-block
}

.sign .overseas .dropdown-menu li a {
  padding: 6px 20px;
  font-size: 14px;
  line-height: 20px
}

.sign .overseas .dropdown-menu li a::hover {
  color: #fff;
  background-color: #f5f5f5
}

.sign .more-sign {
  margin-top: 50px
}

.sign .more-sign h6 {
  position: relative;
  margin: 0 0 10px;
  font-size: 12px;
  color: #b5b5b5
}

.sign .more-sign h6:before {
  left: 30px
}

.sign .more-sign h6:after, .sign .more-sign h6:before {
  content: "";
  border-top: 1px solid #b5b5b5;
  display: block;
  position: absolute;
  width: 60px;
  top: 5px
}

.sign .more-sign h6:after {
  right: 30px
}

.sign .more-sign ul {
  margin-bottom: 10px;
  list-style: none
}

.sign .more-sign ul li {
  margin: 0 5px;
  display: inline-block
}

.sign .more-sign ul a {
  width: 50px;
  height: 50px;
  line-height: 50px;
  display: block
}

.sign .more-sign ul i {
  font-size: 28px
}

.sign .more-sign .ic-weibo {
  color: #e05244
}

.sign .more-sign .ic-wechat {
  color: #00bb29
}

.sign .more-sign .ic-qq_connect {
  color: #498ad5
}

.sign .more-sign .ic-douban {
  color: #00820f
}

.sign .more-sign .ic-more {
  color: #999
}

.sign .more-sign .weibo-loading {
  pointer-events: none;
  cursor: pointer;
  position: relative
}

.sign .more-sign .weibo-loading:after {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: #fff
}

body.reader-night-mode .sign .more-sign .weibo-loading:after {
  background-color: #3f3f3f
}

.sign .more-sign .weibo-loading:before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  margin: -10px 0 0 -10px;
  border-radius: 10px;
  border: 2px solid #e05244;
  border-bottom-color: transparent;
  vertical-align: middle;
  -webkit-animation: rolling .8s infinite linear;
  animation: rolling .8s infinite linear;
  z-index: 1
}

@keyframes rolling {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg)
  }

  to {
    -webkit-transform: rotate(1turn);
    transform: rotate(1turn)
  }
}

@-webkit-keyframes rolling {
  0% {
    -webkit-transform: rotate(0deg)
  }

  to {
    -webkit-transform: rotate(1turn)
  }
}

.sign .reset-password-input {
  border-radius: 4px !important
}

.sign .return {
  margin-left: -8px;
  color: #969696
}

.sign .return:hover {
  color: #333
}

.sign .return i {
  margin-right: 5px
}

.geetest_panel_box > * {
  box-sizing: content-box
}

</style>
