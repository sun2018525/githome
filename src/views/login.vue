<template>
<div class="body">
    <div class="toppanel">
      <h1 class="logo01">电子病历管理系统</h1>
    </div>
    <div class="middlepanel">
      <dl>
        <dt><img src="~@/assets/img/login-img.png" style="margin-top: -50px;" alt="" /></dt>
        <dd>
          <div class="loginContent login-header" >
            <h1><span>用户登录</span></h1>
           <el-form ref="formRef" label-width="80px" :status-icon="true" :model="login" :rules="rules" @keyup.enter="onLogin">
            <el-form-item label-width="0" prop="username">
              <el-input v-model="login.username" placeholder="账号" prefix-icon="user" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label-width="0" prop="password">
              <el-input placeholder="密码" v-model="login.password" prefix-icon="lock" autocomplete="off" show-password></el-input>
            </el-form-item>
            <el-form-item label-width="0" prop="captcha">
              <el-space class="rr-login-right-main-code">
                <el-input v-model="login.captcha" placeholder="验证码" prefix-icon="first-aid-kit"></el-input>
                <img style="vertical-align: middle; height: 40px; cursor: pointer" :src="state.captchaUrl" @click="onRefreshCode" alt="" />
              </el-space>
            </el-form-item>
            <el-form-item label-width="0">
              <el-button type="primary" size="small" :disabled="state.loading" @click="onLogin" class="rr-login-right-main-btn"> 登录 </el-button>
                <span style="color: #000;">没有账号，请先</span>
                <router-link to="/register" >注册</router-link>
             </el-form-item>
          </el-form>
          </div>
        </dd>
      </dl>
    </div> 
  </div>
  
  <!-- <div class="rr-login"> -->
    <!-- <div class="rr-login-wrap">
      <div class="rr-login-left hidden-sm-and-down">
        <p class="rr-login-left-title">电子病历管理系统</p>
      </div> -->

      <!-- <div class="rr-login-right">
        <div class="rr-login-right-main login-header">
          <h4 class="rr-login-right-main-title login-brand">电子病历管理系统</h4>
          <el-form ref="formRef" label-width="80px" :status-icon="true" :model="login" :rules="rules" @keyup.enter="onLogin">
            <el-form-item label-width="0" prop="username">
              <el-input v-model="login.username" placeholder="账号" prefix-icon="user" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label-width="0" prop="password">
              <el-input placeholder="密码" v-model="login.password" prefix-icon="lock" autocomplete="off" show-password></el-input>
            </el-form-item>
            <el-form-item label-width="0" prop="captcha">
              <el-space class="rr-login-right-main-code">
                <el-input v-model="login.captcha" placeholder="验证码" prefix-icon="first-aid-kit"></el-input>
                <img style="vertical-align: middle; height: 40px; cursor: pointer" :src="state.captchaUrl" @click="onRefreshCode" alt="" />
              </el-space>
            </el-form-item>
            <el-form-item label-width="0">
              <el-button type="primary" size="small" :disabled="state.loading" @click="onLogin" class="rr-login-right-main-btn"> 登录 </el-button>
                <span style="color: #000;">没有账号，请先</span>
                <router-link to="/register" >注册</router-link>
             </el-form-item>
          </el-form>
        </div>
      </div>
    </div> -->
    <!-- <div class="login-footer">
      <p><a href="https://www.renren.io/enterprise" target="_blank">企业版</a> | <a href="https://www.renren.io/cloud" target="_blank">微服务版</a></p>
      <p><a href="https://www.renren.io/" target="_blank">人人开源</a>{{ state.year }} © renren.io</p>
    </div> -->
  <!-- </div> -->
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref } from "vue";
import { CacheToken } from "@/constants/cacheKey";
import baseService from "@/service/baseService";
import { setCache } from "@/utils/cache";
import { ElMessage } from "element-plus";
import { getUuid } from "@/utils/utils";
import app from "@/constants/app";
import SvgIcon from "@/components/base/svg-icon/index";
import { useAppStore } from "@/store";
import { useRouter } from "vue-router";
import '@/assets/css/bootstrap.min.less'
import '@/assets/css/font-awesome.min.less'
import '@/assets/css/login.less'
const store = useAppStore();
const router = useRouter();

const state = reactive({
  captchaUrl: "",
  loading: false,
  year: new Date().getFullYear()
});

const login = reactive({ username: "", password: "", captcha: "", uuid: "" });

onMounted(() => {
  //清理数据
  store.logout();
  getCaptchaUrl();
});
const formRef = ref();

const rules = ref({
  username: [{ required: true, message: "用户账号不能为空", trigger: "blur" }],
  password: [{ required: true, message: "密码不能为空", trigger: "blur" }],
  captcha: [{ required: true, message: "验证码不能为空", trigger: "blur" }]
});

const getCaptchaUrl = () => {
  login.uuid = getUuid();
  state.captchaUrl = `${app.api}/captcha?uuid=${login.uuid}`;
};

const onRefreshCode = () => {
  getCaptchaUrl();
};

const onLogin = () => {
  formRef.value.validate((valid: boolean) => {
    if (valid) {
      state.loading = true;
      baseService
        .post("/login", login)
        .then((res) => {
          state.loading = false;
          if (res.code === 0) {
            setCache(CacheToken, res.data, true);
            ElMessage.success("登录成功");
            router.push("/");
          } else {
            ElMessage.error(res.msg);
          }
        })
        .catch(() => {
          state.loading = false;
          onRefreshCode();
        });
    }
  });
};
</script>

<style lang="less" scoped> 
@import url("@/assets/theme/base.less");
.rr-login {
  width: 100vw;
  height: 100vh;
  background-image: url(@/assets/images/login.png);
  background-size: cover; /* 关键：自动铺满且保持比例 */
  min-height: 100vh;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  @media only screen and (max-width: 992px) {
    .rr-login-wrap {
      width: 96% !important;
    }
    .rr-login-right {
      width: 100% !important;
    }
  }
  .login-header {
    padding: 15px;
    color: #409eff;
    .login-brand {
      margin: 0 0 40px;
      font-size: 30px;
      font-weight: 400;
      letter-spacing: 2px;
      text-transform: uppercase;
    }
  }
  &-wrap {
    margin: 0 auto;
    width: 1000px;
    box-shadow: -4px 5px 10px rgba(0, 0, 0, 0.4);
    animation-duration: 1s;
    animation-fill-mode: both;
    border-radius: 5px;
    overflow: hidden;
  }

  &-left {
    justify-content: center;
    flex-direction: column;
    background-color: @--color-primary;
    color: #fff;
    float: left;
    width: 50%;

    &-title {
      text-align: center;
      color: #fff;
      font-weight: 300;
      letter-spacing: 2px;
      font-size: 32px;
    }
  }

  &-right {
    border-left: none;
    color: #fff;
    border-radius :25px;
    background-color: rgba(255, 255, 255, 0.9);
    width: 40%;
    float: left; 
    &-main {
      margin: 0 auto;
      width: 65%;
      &-title {
        color: #333;
        margin-bottom: 40px;
        font-weight: 500;
        font-size: 24px;
        text-align: center;
        letter-spacing: 4px;
      }

      &-lang .iconfont {
        font-size: 20px;
        color: #606266;
        font-weight: 800;
        width: 20px;
        height: 20px;
      }

      .el-input__inner {
        border-width: 0;
        border-radius: 0;
        border-bottom: 1px solid #dcdfe6;
      }

      &-code {
        width: 100%;
        .el-space__item:first-child {
          flex: 1;
        }
      }
      &-btn {
        width: 100%;
        height: 40px;
        font-size: 18px !important;
        letter-spacing: 2px;
        font-weight: 300 !important;
        cursor: pointer;
        font-family: neo, sans-serif;
        transition: 0.25s;
      }
    }
  }
  .rr-login-left{
    background-color: rgba(249, 247, 247, 0.6);
    .rr-login-left-title{
    color: #646568;
    }
  }
  .login-footer {
    text-align: center;
    position: absolute;
    bottom: 0;
    padding: 20px;
    color: rgba(255, 255, 255, 0.6);
    p {
      margin: 10px 0;
    }
    a {
      padding: 0 5px;
      color: rgba(255, 255, 255, 0.6);
      &:focus,
      &:hover {
        color: #fff;
      }
    }
  }

  &-left,
  &-right {
    position: relative;
    min-height: 400px;
    align-items: center;
    display: flex;
  }

  @keyframes animate-down {
    0%,
    60%,
    75%,
    90%,
    to {
      animation-timing-function: cubic-bezier(0.215, 0.61, 0.355, 1);
    }
    0% {
      opacity: 0;
      transform: translate3d(0, -3000px, 0);
    }
    60% {
      opacity: 1;
      transform: translate3d(0, 25px, 0);
    }
    75% {
      transform: translate3d(0, -10px, 0);
    }
    90% {
      transform: translate3d(0, 5px, 0);
    }
    to {
      transform: none;
    }
  }

  .animate-down {
    animation-name: animate-down;
  }
}
</style>