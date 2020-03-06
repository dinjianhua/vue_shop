<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" class="login_form">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            type="password"
            v-model="loginForm.password"
            prefix-icon="iconfont icon-3702mima"
          ></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item>
          <div class="buttons">
            <el-button @click="login" type="primary">登陆</el-button>
            <el-button @click="resetLoginForm" type="info">重置</el-button>
          </div>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单验证规则
      loginFormRules: {
        //   验证用户名是否合法
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 16 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 16, message: '长度在 3 到 16 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate((valid) => {
        if (!valid) return
        this.$axios.post('login', this.loginForm)
          .then((res) => {
            const data = res.data
            if (data.meta.status !== 200) return this.$message.error('登录失败')
            this.$message({ message: '登录成功', type: 'success' })
            /**
             * 1. 将登录成功之后的 token 保存到客户端的 sessionStorage 中
             * 1.1 项目中除了登录之外的其它API接口必须再登陆后才能访问
             * 1.2 token 只应在当前网站打开期间生效，所以将 token 保存在 sessionStorage 中
             */
            window.sessionStorage.setItem('token', data.data.token)
            this.$router.push('/home')
          })
      })
    }
  }
}
</script>

<style lang="less">
.login_container {
  position: relative;
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 5px;
  position: absolute;
  box-sizing: border-box;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avatar_box {
    position: absolute;
    left: 50%;
    background-color: #fff;
    border-radius: 50%;
    transform: translate(-50%, -50%);
    padding: 10px;
    box-shadow: 0 0 6px #aaa;
    width: 120px;
    height: 120px;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
  .login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
  }
  .buttons {
    display: flex;
    justify-content: flex-end;
  }
}
</style>
