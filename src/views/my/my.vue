<template>
  <div class="my-container">
    <!--登录头部-->
    <div v-if="user" class="header user-info">
      <div class="base-info">
        <div class="left">
          <van-image class="avatar" :src="userInfo.photo" round fit="cover"/>
          <span class="name">{{ userInfo.name }}</span>
        </div>
        <div class="right">
          <van-button size="mini" round>编辑资料</van-button>
        </div>
      </div>
      <div class="data-stats">
        <div class="data-item">
          <span class="count">{{ userInfo.art_count }}</span>
          <span class="text">头条</span>
        </div>
        <div class="data-item">
          <span class="count">{{ userInfo.follow_count }}</span>
          <span class="text">关注</span>
        </div>
        <div class="data-item">
          <span class="count">{{ userInfo.fans_count }}</span>
          <span class="text">粉丝</span>
        </div>
        <div class="data-item">
          <span class="count">{{ userInfo.like_count }}</span>
          <span class="text">获赞</span>
        </div>
      </div>
    </div>

    <!--未登录头部-->
    <div v-else class="header not-login">
      <div class="login-btn" @click="$router.push('/login')">
        <img class="mobile-img" src="../../assets/img/mobile.png" alt="">
        <span class="text">登录 / 注册</span>
      </div>
    </div>

    <!--导航-->
    <van-grid class="grid-nav" :column-num="2" clickable>
      <van-grid-item class="grid-item">
        <i slot="icon" class="toutiao toutiao-shoucang"></i>
        <span slot="text" class="text">收藏</span>
      </van-grid-item>
      <van-grid-item class="grid-item">
        <i slot="icon" class="toutiao toutiao-lishi"></i>
        <span slot="text" class="text">历史</span>
      </van-grid-item>
    </van-grid>

    <van-cell title="消息通知" is-link />
    <van-cell class="mb-9" title="小智同学" is-link />
    <van-cell v-if="user" class="logout-cell" title="退出登录" @click="onLogout" clickable/>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import { getUserInfo } from '../../api/user'

  export default {
    name: 'my',
    data() {
      return {
        userInfo: {}
      }
    },
    computed: {
      ...mapState(['user'])
    },
    created() {
      if (this.user) {
        this.loadUserInfo()
      }
    },
    methods: {
      onLogout() {
        // 退出提示
        // 在组件中需要使用 this.$dialog 来调用弹框组件
        this.$dialog.confirm({
          title: '确认退出'
        })
          .then(() => {
            // 确认退出：清除登录状态（容器中的 user + 本地存储中的 user）
            this.$store.commit('setUser', null)
          })
          .catch(() => {
            // on cancel
          })
      },
      async loadUserInfo() {
        try {
          const { data } = await getUserInfo()
          this.userInfo = data.data
        } catch (err) {
          this.$toast('获取数据失败，请稍后重试')
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .my-container {
    .header {
      height: 361px;
      background: url("../../assets/img/banner.png");
      background-size: cover;
    }
    .not-login {
      display: flex;
      justify-content: center;
      align-items: center;
      .login-btn {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        .mobile-img {
          width: 132px;
          height: 132px;
          margin-bottom: 15px;
        }
        .text {
          font-size: 28px;
          color: #fff;
        }
      }
    }
    .user-info {
      .base-info {
        height: 231px;
        box-sizing: border-box;
        padding: 76px 32px 23px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        .left {
          display: flex;
          align-items: center;
          .avatar {
            width: 132px;
            height: 132px;
            margin-right: 23px;
          }
          .name {
            font-size: 30px;
            color: #fff;
          }
        }
      }
      .data-stats {
        display: flex;
        .data-item {
          height: 130px;
          flex: 1;
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          color: #fff;
          .count {
            font-size: 36px;
          }
          .text {
            font-size: 23px;
          }
        }
      }
    }
    .grid-nav {
      .grid-item {
        height: 141px;
        i.toutiao {
          font-size: 45px;
        }
        .toutiao-shoucang {
          color: #eb5253;
        }
        .toutiao-lishi {
          color: #ff9d1d;
        }
        span.text {
          font-size: 28px;
        }
      }
    }
    .logout-cell {
      text-align: center;
      color: #d86262;
    }

    .mb-9 {
      margin-bottom: 9px;
    }
  }
</style>
