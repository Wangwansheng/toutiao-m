<template>
  <div class="login-container">
    <!-- 顶部 -->
    <van-nav-bar
      title="注册 / 登录"
      left-arrow
      @click-left="onClickLeft"
    />
    <!-- 登录表单 -->
    <van-form
    ref="login-form"
    @submit="onLogin"
    :validate-first='true'
    :show-error='false'
    :show-error-message='false'
    @failed='onFailed'
    >
      <van-field
        v-model="user.mobile"
        icon-prefix="toutiao-m"
        left-icon="shouji"
        placeholder="请输入手机号"
        name="mobile"
        :rules="[
          { required: true, message: '请输入手机号' },
          { pattern: /^1[3|5|7|8]\d{9}$/, message: '请输入正确的手机号格式' }
        ]"
      />
      <van-field
        v-model="user.code"
        icon-prefix="toutiao-m"
        left-icon="yanzhengma"
        center
        clearable
        placeholder="请输入短信验证码"
        name="code"
        :rules="[
          { required: true, message: '请输入验证码' },
          { pattern: /^\d{6}$/, message: '请输入正确的验证码格式' }
        ]"
      >
      <!-- 发送验证码 -->
        <template #button>
          <!-- 倒计时 -->
          <van-count-down
          :time="60 * 1000"
          format="ss s"
          v-if="isCountDownShow"
          @finish='isCountDownShow = false'
          />
          <van-button
            class="van-button_code"
            size="small"
            round
            @click.prevent="onSendSms"
            v-else
            :loading="isSendsmsShow"
            loading-text="加载中..."
          >发送验证码</van-button>
        </template>
      </van-field>
      <!-- 登录按钮 -->
      <div style="margin:26px 16px;">
        <van-button block type="info"  @click="onLogin">
          登录
        </van-button>
      </div>
    </van-form>
  </div>
</template>

<script>
import { login, sendSms } from '@/api/user'
import { Toast } from 'vant'

export default {
  name: 'LoginIndex',
  components: {
  },
  props: {
  },
  data () {
    return {
      user: {
        mobile: '',
        code: ''
      },
      // 倒计时的显示隐藏
      isCountDownShow: false,
      isSendsmsShow: false
    }
  },
  computed: {
  },
  watch: {
  },
  created () {
  },
  mounted () {
  },
  methods: {
    async onClickLeft () {
      this.$router.back()
    },
    async onLogin () {
      Toast.loading({
        message: '登录中...',
        forbidClick: true,
        loadingType: 'spinner',
        duration: 0
      })
      // 1 招到数据接口
      // 2 封装请求方法
      // 3 请求调用登录
      try {
        const { data } = await login(this.user)
        // 4 处理响应结果
        Toast.success('登录成功')

        // 将后端返回的用户状态（token等数据）放到Vuex容器中
        this.$store.commit('setUser', data.data)
      } catch (err) {
        console.log(err)
        Toast.fail('登录失败')
      }
    },
    async onFailed (error) {
      if (error.errors[0]) {
        this.$toast({
          message: error.errors[0].message,
          position: 'top'
        })
      }
    },
    // 短信验证码倒计时
    async onSendSms () {
      try {
        this.isSendsmsShow = true
        await this.$refs['login-form'].validate('mobile')
        const res = await sendSms(this.user.mobile)
        this.isCountDownShow = true
        console.log(res)
      } catch (err) {
        let message = ''
        if (err && err.response && err.response.status === 429) {
          message = '发送频繁，请稍后重试'
        } else if (err.name === 'mobile') {
          message = err.message
        } else {
          message = '未知的错误'
        }
        // console.dir(err)
        this.$toast({
          message,
          position: 'top'
        })
      }
      this.isSendsmsShow = false
    }
  }
}
</script>

<style scoped lang="less">
.van-nav-bar{
  background-color: #3296fa;
  /deep/ .van-nav-bar__title{
    font-size: 16px;
    color: #fff;
  }
  /deep/ .van-icon {
    color: #fff;
  }
}
div{
  .van-button{
    font-size: 15px;
    background-color: #6db4fb;
    border: none;
  }
  .van-button_code{
    font-size: 11px;
    background-color: #ccc;
    color: #727272;
  }
}
</style>
