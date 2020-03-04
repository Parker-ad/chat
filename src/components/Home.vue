<template>
  <div class="container">
    <div class="operate">
      <div class="avatar">
        <img :src="avatar" alt="" @click="showEditDialog">
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </div>
    <div class="msgContainer" ref="chatContent">
      <el-card class="box-card">
        <div>
          <span>聊天框</span>
          <el-button style="float: right; padding: 3px 0" type="text" @click="clearMsg">清屏</el-button>
        </div>
        <div v-for="(msgItem, index) in msg" :key="index">
          <ul>
            <li class="once">
              <div class="avatar">
                <img :src="msgItem.avatar" alt="">
                <div class="username"><strong>{{ msgItem.username }}</strong></div>
              </div>
              <div class="text">
                {{ msgItem.text }}
              </div>
            </li>
          </ul>
        </div>
      </el-card>
    </div>
    <div class="textContainer">
      <el-card>
        <el-form>
          <el-form-item>
            <el-input type="textarea" :rows="5" resize="none" placeholder="请输入内容" v-model="text"></el-input>
          </el-form-item>
          <el-form-item class="btn">
            <el-button type="success" @click="sendText">发送</el-button>
          </el-form-item>
        </el-form>
      </el-card>
    </div>
    <el-dialog title="修改信息" :visible.sync="editDialogVisible" width="50%">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item label="邮箱地址" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="editForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="年龄">
          <el-input v-model="editForm.age" type="number"></el-input>
        </el-form-item>
        <el-form-item label="年龄">
          <el-radio v-model="editForm.sex" label="0">男</el-radio>
          <el-radio v-model="editForm.sex" label="1">女</el-radio>
        </el-form-item>
        <el-form-item label="头像">
          <el-upload drag action="http://106.13.52.122:3000/file/upload" name="upload"  :on-success="handleAvatarSuccess" :show-file-list="false" :multiple="false">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
            <div>只能上传'jpg', 'jpeg', 'png', 'gif'文件，且不超过500kb</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
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
      text: '',
      websock: null,
      msg: [],
      username: '',
      avatar: '',
      email: '',
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
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
        ]
      }
    }
  },
  created () {
    this.initWebSocket()
    this.username = window.sessionStorage.getItem('username')
    this.avatar = window.sessionStorage.getItem('avatar')
    this.email = window.sessionStorage.getItem('email')
  },
  // 离开路由之后断开websocket连接
  destroyed () {
    this.websock.close()
  },
  methods: {
    sendText () {
      if (this.text) {
        const username = this.username
        const text = this.text
        const avatar = this.avatar
        const sendedMsg = JSON.stringify({ username, text, avatar })
        this.websocketsend(sendedMsg)
        this.text = ''
      }
    },
    // 初始化weosocket
    initWebSocket () {
      const wsurl = 'ws://106.13.52.122:8080'
      this.websock = new WebSocket(wsurl)
      this.websock.onmessage = this.websocketonmessage
      this.websock.onopen = this.websocketonopen
      this.websock.onerror = this.websocketonerror
      this.websock.onclose = this.websocketclose
    },
    // 连接建立之后执行send方法发送数据
    websocketonopen () {
      const username = this.username
      const text = username + '已上线'
      const avatar = this.avatar
      const sendedMsg = JSON.stringify({ username, text, avatar })
      this.websocketsend(sendedMsg)
    },
    // 连接建立失败重连
    websocketonerror () {
      this.initWebSocket()
    },
    // 数据接收
    websocketonmessage (e) {
      const redata = JSON.parse(e.data)
      this.msg.push(redata)
      this.$refs.chatContent.scrollTop = this.$refs.chatContent.scrollHeight
    },
    // 数据发送
    websocketsend (Data) {
      this.websock.send(Data)
    },
    // 关闭
    websocketclose (e) {
      console.log('断开连接', e)
    },
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    clearMsg () {
      this.msg = []
    },
    async showEditDialog () {
      const { data: res } = await this.$http.get('user/info/' + this.email)
      if (res.err !== 0) return this.$message.error(res.msg)
      this.editForm = res.info
      this.editDialogVisible = true
    },
    async editUserInfo () {
      this.editForm.token = window.sessionStorage.getItem('token')
      const { data: res } = await this.$http.put('user/info', this.editForm)
      if (res.err !== 0) return this.$message.error(res.msg)
      this.$message.success(res.msg)
      this.editDialogVisible = false
    },
    handleAvatarSuccess (res, f, fList) {
      this.editForm.avatar = res.img
      this.avatar = res.img
    }
  }
}
</script>

<style lang="less" scoped>
  .operate {
    height: 100px;
    background-color: #1e1e1e;
    .avatar {
      float: left;
      width: 100px;
      height: 100px;
      border-radius: 25%;
      overflow: hidden;
      img {
        height: 100%;
        cursor: pointer;
      }
    }
    .el-button {
      float: right;
      margin-top: 25px;
      margin-right: 25px;
    }
  }
  .container {
    height: 90%;
    background-color: #2b4b6b;
    .msgContainer {
      overflow:auto;
      height: 70%;
      .once {
        height: 75px;
        border-top: 1px solid #eee;
        color: #000;
        .avatar {
          float: left;
          width: 50px;
          height: 75px;
          padding-top: 10px;
          overflow: hidden;
          img {
            width: 100%;
            border-radius: 50%;
          }
          .username {
            text-align: center;
          }
        }
        .text {
          float: left;
          margin-left: 25px;
          line-height: 75px;
        }
      }
    }
    .textContainer {
      height: 20%;
      border-radius: 20% 20% 0 0;
      .btn {
        display: flex;
        justify-content: end;
      }
    }
  }
</style>
