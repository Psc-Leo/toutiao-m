<template>
  <div class="login-container">
    <!--导航栏-->
    <van-nav-bar class="page-nav-bar" title="登录">
      <van-icon slot="left" name="cross" @click="$router.back()"/>
    </van-nav-bar>
    <!--登录表单-->
    <van-form ref="loginForm" @submit="onSubmit">
      <!--输入框-->
      <van-field name="mobile" placeholder="请输入手机号" v-model="user.mobile" :rules="userFormRules.mobile" type="number" maxlength="11">
        <!--图标需要插槽-->
        <i slot="left-icon" class="toutiao toutiao-shouji"></i>
      </van-field>
      <van-field name="code" placeholder="请输入验证码" v-model="user.code" :rules="userFormRules.code" type="number" maxlength="6">
        <i slot="left-icon" class="toutiao toutiao-yanzhengma"></i>
        <!--发送验证码按钮-->
        <template #button>
          <van-count-down v-if="isCountDownShow" :time="1000 * 60" format="ss s" @finish="isCountDownShow = false"/>
          <van-button v-else class="send-sms-btn" round size="small" type="default" @click="onSendSms" native-type="button">发送验证码</van-button>
        </template>
      </van-field>
      <div class="login-btn-wrap">
        <van-button class="login-btn" block type="info" native-type="submit">登录</van-button>
      </div>
    </van-form>
  </div>
</template>

<script>
  import { login, sendSms } from '../../api/user'

  export default {
    name: 'login',
    data() {
      return {
        user: {
          // 手机号
          mobile: '13911111111',
          // 验证码
          code: '246810'
        },
        userFormRules: {
          mobile: [{
            required: true,
            message: '手机号不能为空'
          }, {
            // rule数据结构中的属性
            pattern: /^1[3|5|7|8]\d{9}$/,
            message: '手机号格式错误'
          }],
          code: [{
            required: true,
            message: '验证码不能为空'
          }, {
            // rule数据结构中的属性
            pattern: /^\d{6}$/,
            message: '验证码格式错误'
          }]
        },
        // 是否展示倒计时
        isCountDownShow: false
      }
    },
    methods: {
      async onSubmit() {
        // 1. 获取表单数据
        const user = this.user
        // 2. 表单验证
        this.$toast.loading({
          message: '登录中...',
          // 禁用背景点击
          forbidClick: true,
          // 持续时间，默认 2000，0 表示持续展示不关闭
          duration: 0
        })
        // 3. 提交表单请求登录
        // 捕获错误
        try {
          const { data } = await login(user)
          this.$store.commit('setUser', data.data)
          this.$toast.success('登录成功')
          this.$router.back()
        } catch (err) {
          if (err.response.status === 400) {
            // console.log('手机号或验证码错误')
            this.$toast.fail('手机号或验证码错误')
          } else {
            // console.log('登录失败，请稍后重试', err)
            this.$toast.fail('登录失败，请稍后重试')
          }
        }
        // 4. 根据请求响应结果处理后续操作
      },
      async onSendSms() {
        // 1. 校验手机号
        try {
          await this.$refs.loginForm.validate('mobile')
        } catch (err) {
          return console.log('验证失败', err)
        }
        // 2. 验证通过，显示倒计时
        this.isCountDownShow = true
        // 3. 请求发送验证码
        try {
          await sendSms(this.user.mobile)
          this.$toast('发送成功')
        } catch (err) {
          // 发送失败，关闭倒计时
          this.isCountDownShow = false
          if (err.response.status === 429) {
            this.$toast('发送太频繁了，请稍后重试')
          } else {
            this.$toast('发送失败，请稍后重试')
          }
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .login-container {
    .toutiao {
      font-size: 37px;
    }
    .send-sms-btn {
      width: 152px;
      height: 46px;
      line-height: 46px;
      font-size: 22px;
      background-color: #ededed;
      color: #666;
    }
    .login-btn-wrap {
      padding: 53px 33px;
      .login-btn {
        background-color: #6db4fb;
        border: none;
      }
    }
  }
</style>
