<template>
  <el-form class="login-form" status-icon :rules="loginRules" ref="loginForm" :model="loginForm" label-width="0">
    <el-form-item prop="mobile">
      <el-input  @keyup.enter.native="handleLogin" v-model="loginForm.mobile" auto-complete="off" placeholder="请输入手机号码">
        <svg-icon slot="prefix" icon-class="mobile" class="el-input__icon input-icon"/>
      </el-input>
    </el-form-item>
    <el-form-item prop="code">
      <el-input @keyup.enter.native="handleLogin" v-model="loginForm.code" auto-complete="off" placeholder="请输入验证码">
        <svg-icon slot="prefix" icon-class="validCode" class="el-input__icon input-icon"/>
        <template slot="append">
          <span @click="handleSend" class="msg-text" :class="[{display:msgKey}]">{{msgText}}</span>
        </template>
      </el-input>
    </el-form-item>
    <el-form-item>
      <el-button size="medium" type="primary" @click.native.prevent="handleLogin"  style="width:100%;">
        登录
      </el-button>
    </el-form-item>
  </el-form>
</template>

<script>
  const MSGINIT = '发送验证码'
  // const MSGERROR = '验证码发送失败'
  const MSGSCUCCESS = '${time}秒后重发'
  const MSGTIME = 60
  import { validateMobile } from '@/utils/validate'
  import {getMobileCode} from "@/api/login";

  export default {
    name: 'mobileLogin',
    data() {
      const validatemobile = (rule, value, callback) => {
        if (validateMobile(value)[0]) {
          callback(new Error(validateMobile(value)[1]))
        } else {
          callback()
        }
      }
      const validateCode = (rule, value, callback) => {
        if (value.length !== 4) {
          callback(new Error('请输入4位数的验证码'))
        } else {
          callback()
        }
      }
      return {
        msgText: MSGINIT,
        msgTime: MSGTIME,
        msgKey: false,
        loginForm: {
          mobile: '18810001000',
          code: ''
        },
        loginRules: {
          mobile: [{ required: true, trigger: 'blur', validator: validatemobile }],
          code: [{ required: true, trigger: 'blur', validator: validateCode }]
        }
      }
    },
    created() {},
    mounted() {},
    computed: {
    },
    props: [],
    methods: {
      handleSend() {

        getMobileCode(this.loginForm.mobile).then(response => {
          if (response.data.data) {
            this.$message.success('验证码发送成功')
          } else {
            this.$message.error(response.data.msg)
          }
        })
        if (this.msgKey) return
        this.msgText = MSGSCUCCESS.replace('${time}', this.msgTime)
        this.msgKey = true
        const time = setInterval(() => {
          this.msgTime--
          this.msgText = MSGSCUCCESS.replace('${time}', this.msgTime)
          if (this.msgTime === 0) {
            this.msgTime = MSGTIME
            this.msgText = MSGINIT
            this.msgKey = false
            clearInterval(time)
          }
        }, 1000)
      },
      handleLogin() {
        this.$refs.loginForm.validate(valid => {
          if (valid) {
            this.$store.dispatch('user/loginByMobile', this.loginForm).then(res => {
              this.$router.push({ path: '/' })
            })
          }
        })
      }
    }
  }
</script>

<style>
  .code{
    display:block;
    margin-top: 8px;
  }
  .msg-text {
    display: block;
    width: 60px;
    font-size: 12px;
    text-align: center;
    cursor: pointer;
  }
  .msg-text.display {
    color: #ccc;
  }
</style>
