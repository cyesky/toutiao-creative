<template>
  <div class="login-container">
    <div class="login-box">
      <!--ref:必须属性，后期方便el-form组件获取使用-->
      <!--model:必须属性，与v-model没有关系，对表单全部数据对象进行绑定，后期表单校验会使用到-->
      <!--label-width:非必须属性，表单项目名称宽度-->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
        <img src="./logo_index.png" alt="">
        <!--el-form-item:是表单项目组件，每个表单域需要通过此组件圈选-->
        <!--label:非必须属性，表单项目名称-->
        <!--prop:使得校验规则与当前项目联系起来，注意：属性值必须是表单对象成员名称-->
        <el-form-item prop="mobile">
          <!--el-input:普通输入框组件-->
          <!--v-model:必须属性，双向数据绑定-->
          <el-input v-model="loginForm.mobile" placeholder="请输入手机号码">
            <i slot="prefix" class="iconfont icon-shoujihao"></i>
          </el-input>
        </el-form-item>
        <el-form-item prop="code">
          <el-input v-model="loginForm.code" placeholder="请输入校验码">
            <i slot="prefix" class="iconfont icon-yanzhengma"></i>
          </el-input>
        </el-form-item>
        <el-form-item prop="xieyi" style="text-align:left;">
          <el-checkbox v-model="loginForm.xieyi" style="margin-right:10px;"></el-checkbox>
          <span>我已阅读并同意用户协议和隐私条款</span>
        </el-form-item>
        <el-form-item>
          <el-button :loading="isActive" :disabled="isActive" type="primary" style="width:100%;" @click="login()">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
// 引入极验的gt.js文件
// 该gt.js内部没有export default导出语句
// 该文件内部有通过window声明全局变量，可以给当前应用做使用
import '../../assets/js/gt.js'

import store from '@/store'

// 引入iconfont的样式文件
import '../../assets/iconfont/iconfont.css'

export default {
  data () {
    // 自定义校验协议的方法
    var xieyiTest = function (rule, value, callback) {
      // rule:当前项目具体校验规则，一般不用
      // value:当前项目的值信息
      // callback:回调函数，校验成功或失败都要执行
      // if (value) {
      //   callback()
      // } else {
      //   callback(new Error('无条件遵守我们的规则'))
      // }
      // 升级代码
      value ? callback() : callback(new Error('无条件遵守我们的规则'))
    }
    return {
      isActive: false, // 登录按钮等待、禁用状态
      ctaObj: null, // 极验窗口对象
      loginForm: {
        mobile: '13911111111', // 手机号码
        code: '246810', // 校验码
        xieyi: true
      },
      // 登录校验
      loginFormRules: {
        // 校验字段: [{ 规则 }, { 规则 }, { 规则 }]
        mobile: [
          { required: true, message: '手机号码必填' },
          { pattern: /^1[35789]\d{9}$/, message: '手机号码遵守格式' }
        ],
        code: [
          { required: true, message: '校验码必填' }
        ],
        xieyi: [
          { validator: xieyiTest }
        ]
      }
    }
  },
  methods: {
    // 登录系统
    login () {
      // 全部表单域项目校验
      // 获得form表单组件的语句: this.$refs.loginFormRef
      // form组件.validate(回调函数)
      // 参数valid：会体现布尔值，表示校验是否成功
      this.$refs.loginFormRef.validate((valid) => {
        if (valid) {
          // 表单校验成功
          // 判断极验窗口存在就直接使用
          if (this.ctaObj !== null) {
            return this.ctaObj.verify() // 显示窗口
          }

          // 登录按钮禁用、等待
          this.isActive = true

          this.loginAct()

          // A. 人机交互验证
          //    获得人机交互验证的秘钥信息
          // let pro = this.$http.get(`/captchas/${this.loginForm.mobile}`)
          // pro
          //   .then(result => {
          //     // 对象解构赋值
          //     let { data } = result.data
          //     // axios请求成功
          //     // 生成极验验证"窗口"
          //     // 请检测data的数据结构， 保证data.gt, data.challenge, data.success有值
          //     window.initGeetest({
          //       // 以下配置参数来自服务端 SDK
          //       gt: data.gt,
          //       challenge: data.challenge,
          //       offline: !data.success,
          //       new_captcha: true,
          //       product: 'bind' // 隐藏按钮，通过登录按钮激活
          //     }, captchaObj => {
          //       // 这里可以调用验证实例 captchaObj 的实例方法
          //       captchaObj.onReady(() => {
          //         // 验证码ready之后才能调用verify方法显示验证码
          //         // 生成窗口
          //         captchaObj.verify()
          //         // 把窗口对象赋予给ctaObj对象
          //         this.ctaObj = captchaObj
          //         // 登录按钮状态恢复
          //         this.isActive = false
          //       }).onSuccess(() => {
          //         // 人的行为正确，登录后台系统
          //         // B. axios发送请求进行账号“真实性校验”，登录后台
          //         this.loginAct()
          //       }).onError(() => {
          //         // your code
          //       })
          //     })
          //   })
          //   .catch(err => {
          //     return this.$message.error('获得人机秘钥信息有错误：' + err)
          //   })
        }
      })
    },
    // 账号真实校验，登录后台页面
    async loginAct () {
      try {
        const res = await this.$http.post('authorizations', this.loginForm)
        store.setUser(res.data.data)
        this.$router.push('/')
      } catch (e) {
        this.isActive = false
        this.$message.error('手机号或验证码错误')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.login-container {
  background-color: gray;
  background-size:cover;
  background-image: url(./login_bg.jpg);
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  .login-box {
    width: 400px;
    height: 340px;
    background-color: white;
    display:flex;
    justify-content: center;
    align-items: center;
    text-align:center;
    img{
      width:50%;
      margin:20px auto;
    }
    .el-form{
      width:75%;
    }
  }
}
</style>
