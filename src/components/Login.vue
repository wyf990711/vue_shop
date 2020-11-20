<template>
<div class="login_container">
  <div class="login_box">
    <div class="avatar_box">
      <img src="../assets/login.png" alt="">
    </div>
    <!--登录表单区 -->
    <el-form ref="loginFormRef" :model="loginform" :rules="loginFormRules" label-width="0px" class="login_form">
      <!--用户名 -->
      <el-form-item prop="username">
        <el-input v-model="loginform.username" prefix-icon="iconfont icon-user"></el-input>
      </el-form-item>
      <!--密码 -->
      <el-form-item prop="password">
        <el-input v-model="loginform.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input>
      </el-form-item>
      <!--按钮区域-->
      <el-form-item class="btns">
        <el-button type="primary" @click="login">登录</el-button>
        <el-button type="info" @click="resetLoginFrom">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</div>
</template>

<script>
export default {
  data() {
    return {
      // 这是登录表单的数据绑定对象
      loginform: {
        username: 'admin',
        password: '123456'
      },
      //这是表单的验证规则对象
      loginFormRules: {
        //验证用户名
        username: [{
            required: true,
            message: '请输入登录名称',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 10,
            message: '长度在3到10个字符',
            trigger: 'blur'
          }
        ],
        //验证密码
        password: [{
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 15,
            message: '长度在6到15个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    //点击重置按钮。重置登录表单
    resetLoginFrom() {
      this.$refs.loginFormRef.resetFields();
    },
    login() {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return;
        const {
          data: res
        } = await this.$http.post('login', this.loginform);
        if (res.meta.status !== 200) return this.$message.error('登录失败');
        this.$message.success('登录成功');
        // 1.将登录成功之后的token，保持到客户端的sessionStorage中
        //  项目中除了登录之外的其他API接口，必须在登录之后才能访问
        //  token只应在当前网站打开期间生效，所以保存在sessionStorage中
        // console.log(res);
        // 将token保存到sessionStorage
        window.sessionStorage.setItem('token', res.data.token);
        //通过编程式导航跳转到后台主页。路由地址是/Home
        this.$router.push('/home')
      })
    }
  }

}
</script>

<style lang="less">
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 5px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);

  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eeeeee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #dddddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;

    img {
      height: 100%;
      width: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}

.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>>
