<template>
  <div class="login-body">
    <div class="bg"></div>
    <div class="login-panel">
      <el-form
          class="login-register"
          :model="formData"
          :rules="rules"
          ref="formDataRef"
      >
        <div class="login-title">SmallPan | 云盘系统</div>
        <!--input输入-->
        <el-form-item prop="email">
          <el-input
              size="large"
              clearable
              placeholder="请输入邮箱"
              v-model="formData.email"
              maxLength="150"
          >
            <template #prefix>
              <span class="iconfont icon-account"></span>
            </template>
          </el-input>
        </el-form-item>
        <!--登录密码-->
        <el-form-item prop="password" v-if="opType == 1">
          <el-input
              type="password"
              size="large"
              placeholder="请输入密码"
              v-model="formData.password"
              show-password
          >
            <template #prefix>
              <span class="iconfont icon-password"></span>
            </template>
          </el-input>
        </el-form-item>
        <!--注册-->
        <div v-if="opType == 0 || opType == 2">
          <el-form-item prop="emailCode">
            <div class="send-emali-panel">
              <el-input
                  size="large"
                  placeholder="请输入邮箱验证码"
                  v-model="formData.emailCode"
              >
                <template #prefix>
                  <span class="iconfont icon-checkcode"></span>
                </template>
              </el-input>
              <el-button
                  class="send-mail-btn"
                  type="primary"
                  size="large"
                  @click="getEmailCode"
              >获取验证码
              </el-button
              >
            </div>
            <el-popover placement="left" :width="500" trigger="click">
              <div>
                <p>在垃圾箱中查找邮箱验证码</p>
              </div>
              <template #reference>
                <span class="a-link" :style="{ 'font-size': '14px' }"
                >未收到邮箱验证码？</span
                >
              </template>
            </el-popover>
          </el-form-item>
          <el-form-item prop="nickName" v-if="opType == 0">
            <el-input
                size="large"
                clearable
                placeholder="请输入昵称"
                v-model="formData.nickName"
                maxLength="20"
            >
              <template #prefix>
                <span class="iconfont icon-account"></span>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="registerPassword">
            <el-input
                type="password"
                size="large"
                placeholder="请输入密码"
                v-model="formData.registerPassword"
                show-password
            >
              <template #prefix>
                <span class="iconfont icon-password"></span>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="reRegisterPassword">
            <el-input
                type="password"
                size="large"
                placeholder="请再次输入密码"
                v-model="formData.reRegisterPassword"
                show-password
            >
              <template #prefix>
                <span class="iconfont icon-password"></span>
              </template>
            </el-input>
          </el-form-item>
        </div>
        <el-form-item prop="checkCode">
          <div class="check-code-panel">
            <el-input
                size="large"
                placeholder="请输入验证码"
                v-model="formData.checkCode"
                @keyup.enter="doSubmit"
            >
              <template #prefix>
                <span class="iconfont icon-checkcode"></span>
              </template>
            </el-input>
            <img
                :src="checkCodeUrl"
                class="check-code"
                @click="changeCheckCode(0)"
            />
          </div>
        </el-form-item>
        <el-form-item v-if="opType == 1">
          <div class="rememberme-panel">
            <a href="javascript:void(0)" class="a-link" @click="showPanel(2)"
            >忘记密码？</a
            >
          </div>
          <div class="no-account">

            <a href="javascript:void(0)" class="a-link" @click="showPanel(0)"
            >没有账号？</a
            >
          </div>
        </el-form-item>
        <el-form-item v-if="opType == 0">
          <a href="javascript:void(0)" class="a-link" @click="showPanel(1)"
          >已有账号?</a
          >
        </el-form-item>
        <el-form-item v-if="opType == 2">
          <a href="javascript:void(0)" class="a-link" @click="showPanel(1)"
          >去登录?</a
          >
        </el-form-item>
        <el-form-item>
          <el-button
              type="primary"
              class="op-btn"
              @click="doSubmit"
              size="large"
          >
            <span v-if="opType == 0">注册</span>
            <span v-if="opType == 1">登录</span>
            <span v-if="opType == 2">重置密码</span>
          </el-button>
        </el-form-item>
<!--        <div class="login-btn-qq" v-if="opType == 1">-->
<!--          Gitee <img src="@/assets/icon-image/gitee.png" @click="qqLogin"/>-->
<!--        </div>-->
      </el-form>
    </div>
    <!--发送邮箱验证码-->
    <Dialog
        :show="dialogConfig4SendMailCode.show"
        :title="dialogConfig4SendMailCode.title"
        :buttons="dialogConfig4SendMailCode.buttons"
        width="500px"
        :showCancel="false"
        @close="dialogConfig4SendMailCode.show = false"
    >
      <el-form
          :model="formData4SendMailCode"
          :rules="rules"
          ref="formData4SendMailCodeRef"
          label-width="80px"
      >
        <el-form-item label="邮箱">
          {{ formData.email }}
        </el-form-item>
        <el-form-item label="验证码" prop="checkCode">
          <div class="check-code-panel">
            <el-input
                size="large"
                placeholder="请输入验证码"
                v-model="formData4SendMailCode.checkCode"
            >
              <template #prefix>
                <span class="iconfont icon-checkcode"></span>
              </template>
            </el-input>
            <img
                :src="checkCodeUrl4SendMailCode"
                class="check-code"
                @click="changeCheckCode(1)"
            />
          </div>
        </el-form-item>
      </el-form>
    </Dialog>
  </div>
</template>

<script setup>
import {ref, reactive, getCurrentInstance, nextTick, onMounted} from "vue";
import {useRouter, useRoute} from "vue-router";
import md5 from "js-md5";

const {proxy} = getCurrentInstance();
const router = useRouter();
const route = useRoute();
const api = {
  checkCode: "/api/checkCode",
  sendMailCode: "/sendEmailCode",
  register: "/register",
  login: "/login",
  resetPwd: "/resetPwd",
  qqlogin: "/qqlogin",
};

// 0:注册 1:登录 2:重置密码
const opType = ref();
const showPanel = (type) => {
  opType.value = type;
  resetForm();
};

onMounted(() => {
  showPanel(1);
});

//验证码
const checkCodeUrl = ref("?type=0");
const checkCodeUrl4SendMailCode = ref("?type=1");
const changeCheckCode = (type) => {
  if (type == 0) {
    checkCodeUrl.value =
        api.checkCode + "?type=" + type + "&time=" + new Date().getTime();
  } else {
    checkCodeUrl4SendMailCode.value =
        api.checkCode + "?type=" + type + "&time=" + new Date().getTime();
  }
};

//发送邮箱验证码弹窗
const formData4SendMailCode = ref({});
const formData4SendMailCodeRef = ref();
const dialogConfig4SendMailCode = reactive({
  show: false,
  title: "发送邮箱验证码",
  buttons: [
    {
      type: "primary",
      text: "发送验证码",
      click: () => {
        sendEmailCode();
      },
    },
  ],
});
//获取邮箱验证码
const getEmailCode = () => {
  formDataRef.value.validateField("email", (valid) => {
    if (!valid) {
      return;
    }
    dialogConfig4SendMailCode.show = true;

    nextTick(() => {
      changeCheckCode(1);
      formData4SendMailCodeRef.value.resetFields();
      formData4SendMailCode.value = {
        email: formData.value.email,
      };
    });
  });
};
//发送邮件
const sendEmailCode = async () => {
  formData4SendMailCodeRef.value.validate(async (valid) => {
    if (!valid) {
      return;
    }
    const params = Object.assign({}, formData4SendMailCode.value);
    params.type = opType.value == 0 ? 0 : 1;
    let result = await proxy.Request({
      url: api.sendMailCode,
      params: params,
      errorCallback: () => {
        changeCheckCode(1);
      },
    });
    proxy.Message.success("验证码发送成功，请登录邮箱查看");
    dialogConfig4SendMailCode.show = false;
  });
};

//登录，注册 弹出配置
const dialogConfig = reactive({
  show: false,
  title: "标题",
});

const checkRePassword = (rule, value, callback) => {
  if (value !== formData.value.registerPassword) {
    callback(new Error(rule.message));
  } else {
    callback();
  }
};
const formData = ref({});
const formDataRef = ref();
const rules = {
  email: [
    {required: true, message: "请输入邮箱"},
    {validator: proxy.Verify.email, message: "请输入正确的邮箱"},
  ],
  password: [{required: true, message: "请输入密码"}],
  emailCode: [{required: true, message: "请输入邮箱验证码"}],
  nickName: [{required: true, message: "请输入昵称"}],
  registerPassword: [
    {required: true, message: "请输入密码"},
    {
      validator: proxy.Verify.password,
      message: "密码只能是数字，字母，特殊字符 8-18位",
    },
  ],
  reRegisterPassword: [
    {required: true, message: "请再次输入密码"},
    {
      validator: checkRePassword,
      message: "两次输入的密码不一致",
    },
  ],
  checkCode: [{required: true, message: "请输入图片验证码"}],
};

//重置表单
const resetForm = () => {
  dialogConfig.show = true;
  if (opType.value == 0) {
    dialogConfig.title = "注册";
  } else if (opType.value == 1) {
    dialogConfig.title = "登录";
  } else if (opType.value == 2) {
    dialogConfig.title = "重置密码";
  }
  nextTick(() => {
    changeCheckCode(0);
    formDataRef.value.resetFields();
    formData.value = {};

    //登录
    if (opType.value == 1) {
      const cookieLoginInfo = proxy.VueCookies.get("loginInfo");
      if (cookieLoginInfo) {
        formData.value = cookieLoginInfo;
      }
    }
  });
};

// 登录、注册、重置密码  提交表单
const doSubmit = () => {
  formDataRef.value.validate(async (valid) => {
    if (!valid) {
      return;
    }
    let params = {};
    Object.assign(params, formData.value);
    //注册
    if (opType.value == 0 || opType.value == 2) {
      params.password = md5(params.registerPassword);
      delete params.registerPassword;
      delete params.reRegisterPassword;
    }
    //登录
    if (opType.value == 1) {
      let cookieLoginInfo = proxy.VueCookies.get("loginInfo");
      let cookiePassword =
          cookieLoginInfo == null ? null : cookieLoginInfo.password;
      if (params.password !== cookiePassword) {
        params.password = md5(params.password);
      }
    }
    let url = null;
    if (opType.value == 0) {
      url = api.register;
    } else if (opType.value == 1) {
      url = api.login;
    } else if (opType.value == 2) {
      url = api.resetPwd;
    }
    let result = await proxy.Request({
      url: url,
      params: params,
      errorCallback: () => {
        changeCheckCode(0);
      },
    });
    if (!result) {
      return;
    }
    //注册返回
    if (opType.value == 0) {
      proxy.Message.success("注册成功,请登录");
      showPanel(1);
    } else if (opType.value == 1) {
      //登录
      if (params.rememberMe) {
        const loginInfo = {
          email: params.email,
          password: params.password,
          rememberMe: params.rememberMe,
        };
        proxy.VueCookies.set("loginInfo", loginInfo, "7d");
      } else {
        proxy.VueCookies.remove("loginInfo");
      }
      dialogConfig.show = false;
      proxy.Message.success("登录成功");
      //存储cookie
      proxy.VueCookies.set("userInfo", result.data, 0);
      //重定向到原始页面
      const redirectUrl = route.query.redirectUrl || "/";
      router.push(redirectUrl);
    } else if (opType.value == 2) {
      //重置密码
      proxy.Message.success("重置密码成功,请登录");
      showPanel(1);
    }
  });
};

const closeDialog = () => {
  dialogConfig.show = false;
};

//QQ登录
const qqLogin = async () => {
  let result = await proxy.Request({
    url: api.qqlogin,
    params: {
      callbackUrl: route.query.redirectUrl || "",
    },
  });
  if (!result) {
    return;
  }
  proxy.VueCookies.remove("userInfo");
  document.location.href = result.data;
};
</script>

<style lang="scss" scoped>
.login-body {
  height: 100vh;
  background: url("../assets/background2.jpg") center/cover no-repeat;
  display: flex;
  justify-content: center;
  align-items: center;

  .login-panel {
    width: 400px;
    background: rgba(255, 255, 255, 0.85); /* 半透明背景，增强对比度 */
    border-radius: 8px;
    padding: 30px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    text-align: center;

    .login-title {
      font-size: 22px;
      font-weight: 600;
      color: #333;
      margin-bottom: 20px;
    }

    .login-register {
      .send-emali-panel {
        display: flex;
        justify-content: space-between;
        margin-bottom: 15px;

        .send-mail-btn {
          background-color: #007FFF;
          border: none;
          color: white;
          padding: 10px 15px;
          font-size: 14px;
          border-radius: 5px;
          transition: background-color 0.3s ease;
          &:hover {
            background-color: #007FFF;
          }
        }
      }

      .rememberme-panel {
        display: flex;
        align-items: center;
        justify-content: flex-start;
        margin-top: 10px;
      }

      .no-account {
        display: flex;
        justify-content: right;
        margin-left: auto;

        .a-link {
          color: #007FFF;
          font-size: 14px;
          text-decoration: none;
          &:hover {
            color: #007FFF;
            text-decoration: underline;
          }
        }
      }

      .op-btn {
        width: 100%;
        margin-top: 20px;
        padding: 12px 0;
        background-color: #007FFF;
        color: white;
        font-size: 16px;
        font-weight: bold;
        border-radius: 5px;
        border: none;
        cursor: pointer;
        transition: background-color 0.3s ease;
        &:hover {
          background-color: #007FFF;
        }
      }
    }
  }

  .check-code-panel {
    display: flex;
    align-items: center;
    justify-content: space-between;

    .check-code {
      margin-left: 10px;
      cursor: pointer;
      border-radius: 5px;
      transition: transform 0.3s ease;
      &:hover {
        transform: scale(1.1);
      }
    }
  }

  .login-btn-qq {
    margin-top: 30px;
    display: flex;
    justify-content: center;
    align-items: center;

    img {
      cursor: pointer;
      width: 30px;
      transition: transform 0.3s ease;
      &:hover {
        transform: scale(1.2);
      }
    }
  }
}


</style>