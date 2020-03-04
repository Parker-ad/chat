<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/1.jpg" alt="">
      </div>
      <!-- 登录表单区域 -->
      <el-form label-width="0px" class="login_form" ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
        <!-- 用户名 -->
        <el-form-item prop="email">
          <el-input v-model="loginForm.email" prefix-icon="el-icon-user-solid" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" prefix-icon="el-icon-lock" type="password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button type="success" @click="regDialogVisible = true">注册</el-button>
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!-- 注册对话框 -->
    <el-dialog title="注册" :visible.sync="regDialogVisible" width="50%" @close="resetRegForm">
      <el-form :model="regForm" :rules="regFormRules" ref="regFormRef" label-width="100px">
        <el-form-item label="邮箱地址" prop="email">
          <el-input v-model="regForm.email"></el-input>
        </el-form-item>
        <el-form-item label="邮箱验证码" prop="code">
          <el-input v-model="regForm.code">
            <el-button slot="append" @click="sendCode">获取验证码</el-button>
          </el-input>
        </el-form-item>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="regForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="regForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="年龄">
          <el-input v-model="regForm.age" type="number"></el-input>
        </el-form-item>
        <el-form-item label="年龄">
          <el-radio v-model="regForm.sex" label="0">男</el-radio>
          <el-radio v-model="regForm.sex" label="1">女</el-radio>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="info" @click="resetRegForm">重置</el-button>
        <el-button @click="regDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="regist">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([A-z0-9_-])+@([A-z0-9_-])+(\.[A-z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法
        return cb()
      }
      // 不合法
      cb(new Error('请输入合法的邮箱'))
    }
    return {
      // 这是登录表单的数据绑定对象
      loginForm: {
        email: '',
        password: ''
      },
      // 这是表单的验证规则对象
      loginFormRules: {
        // 验证用户名是否合法
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在6到15个字符', trigger: 'blur' }
        ]
      },
      regDialogVisible: false,
      regForm: {
        email: '',
        username: '',
        password: '',
        age: 18,
        sex: '0',
        code: ''
      },
      regFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 6, message: '长度在3到6个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在6到15个字符', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 点击充值按钮 重置登录表单
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        // 如果验证没通过验证规则直接退出
        if (!valid) return
        // 如果通过验证则发送请求
        const { data: res } = await this.$http.post('user/login', this.loginForm)
        if (res.err !== 0) return this.$message.error(res.msg)
        this.$message.success(res.msg)
        window.sessionStorage.setItem('token', res.token)
        window.sessionStorage.setItem('username', res.username)
        window.sessionStorage.setItem('avatar', res.avatar)
        window.sessionStorage.setItem('email', res.email)
        // 编程式导航跳转到/home路径
        this.$router.push('/home')
      })
    },
    resetRegForm () {
      this.$refs.regFormRef.resetFields()
    },
    async sendCode () {
      if (!this.regForm.email) return this.$message.error('请输入邮箱地址后获取验证码')
      const { data: res } = await this.$http.post('user/mailcode', { email: this.regForm.email })
      this.$message.success(res.msg)
    },
    regist () {
      this.$refs.regFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('user/reg', this.regForm)
        if (res.err !== 0) return this.$message.error(res.msg)
        this.$message.success(res.msg)
        this.regDialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  height: 100%;
  background-color: #2b4b6b;
}
.login_box {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  transform: translate(-50%, -50%);

  .avatar_box {
    position: absolute;
    left: 50%;
    width: 130px;
    height: 130px;
    padding: 10px;
    background-color: #fff;
    border: 1px solid #eee;
    border-radius: 50%;
    box-shadow: 0 0 10px #ddd;
    transform: translate(-50%, -50%);
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
      }
    }

  .login_form {
    box-sizing: border-box;
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 20px;
    .btns {
        display: flex;
        justify-content: flex-end;
        }
    }
  }
</style>
