<template>
  <div class="reg-container">
    <!--头部 -->
    <el-row class="reg-title">
      <!-- 文字 -->
      <el-row class="reg-titile-text">
        <span class="line" />&nbsp;&nbsp;欢迎注册电子病历系统&nbsp;&nbsp;
        <span class="line" />
      </el-row>
    </el-row>
    <!-- 注册框 -->
    <el-row class="regForm">
      <el-row class="regForm-text">
        <el-form ref="regFormRef" :model="regForm" :rules="rules">
          <el-col :span="16" class="textinfo1">
            <el-form-item class="labelm" label="手 机 号" prop="mobile" label-width="150px">
              <el-input v-model="regForm.mobile" name="mobile" size="large" placeholder="请输入手机号" />
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="用户账号" prop="username" label-width="150px">
              <el-input v-model="regForm.username" size="large" placeholder="该账号用作登录" />
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="姓   名" prop="realName" label-width="150px">
              <el-input v-model="regForm.realName" size="large" placeholder="请输入姓名" />
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="身份证号" prop="idCard" label-width="150px">
              <el-input v-model="regForm.idCard" size="large" placeholder="请输入身份证号" />
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="密码" prop="password" label-width="150px">
              <el-input v-model="regForm.password" type="password" size="large" placeholder="请输入密码" />
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="确认密码" prop="confirmPassword" label-width="150px">
              <el-input v-model="regForm.confirmPassword" type="password" size="large" placeholder="请再次确认密码" />
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="性别" prop="gender" label-width="150px">
              <el-radio-group v-model="regForm.gender" size="large" placeholder="请输入性别">
                <el-radio :label="0">男</el-radio>
                <el-radio :label="1">女</el-radio> 
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="头像" prop="headUrl" label-width="150px">
              <el-upload
                class="avatar-uploader"
                :action="headPicUrl"
                :show-file-list="false"
                drag
                :on-success="successHandle"
                :before-upload="beforeUploadHandle"
              >
              <img v-if="regForm.headUrl" :src="headPicSrc" class="avatar" />
              <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
              </el-upload>
              <div slot="tip" class="el-upload__tip">
                只能上传jpg、jpeg、png、gif文件，且不超过2M
              </div> 
            </el-form-item>
          </el-col>
          <el-col :span="16" class="textinfo">
            <el-form-item class="labelm" label="电子邮箱" prop="email" label-width="150px">
              <el-input v-model="regForm.email" size="large" placeholder="请输入电子邮箱" />
            </el-form-item>
          </el-col>
          <el-col class="regBtn" :span="16">
            <el-button size="medium" class="reg" type="primary" @click="regSubmit">注&nbsp; 册</el-button>
          </el-col>
            <el-col class="regBtn" :span="16">
            已有账号？<router-link to="/login">登录</router-link>
            <router-view></router-view>
       </el-col>
        </el-form>
      </el-row>
     
    </el-row>
  </div>
</template>
<script lang="ts" setup>

import { reactive, ref } from "vue";
import { isEmail, isMobile ,isIdCard} from "@/utils/utils";
import { ElMessage } from "element-plus";
import baseService from "@/service/baseService";
import { useRouter } from "vue-router";
import type { UploadProps } from 'element-plus'
import app from "@/constants/app";

const router = useRouter();
const regFormRef = ref();
const headPicUrl = ref(`${app.api}/registerUpload`);
const headPicSrc = ref();

const successHandle: UploadProps['onSuccess'] = (
  response,
  uploadFile
) => {
  if (response.code !== 0) {
    return ElMessage.error(response.msg);
  }
  headPicSrc.value = URL.createObjectURL(uploadFile.raw!);
  regForm.headUrl="/headUrl/"+uploadFile.name;
}

const beforeUploadHandle: UploadProps['beforeUpload'] = (file) => {
  if (file.type !== "image/jpg" && file.type !== "image/jpeg" && file.type !== "image/png" && file.type !== "image/gif") {
    ElMessage.error("只支持jpg、jpeg、png、gif格式文件！");
    return false;
  } else if (file.size / 1024 / 1024 > 2) {
    ElMessage.error('头像大小不能超过2M!')
    return false
  }
  return true
}
const regForm = reactive(
  { mobile: "",username:"",realName:"", password: "", confirmPassword: "", gender: "",
    idCard:"",headUrl:"",email:""
   },
);

const validatePassword = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (!/\S/.test(value)) {
    return callback(new Error("密码不能为空"));
  }
  callback();
};
const validateConfirmPassword = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (!/\S/.test(value)) {
    return callback(new Error("确认密码不能为空"));
  }
  if (regForm.password !== value) {
    return callback(new Error("确认密码与密码输入不一致"));
  }
  callback();
};
const validateEmail = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (!/\S/.test(value)) {
    return callback(new Error("邮箱不能为空"));
  }
  if (value && !isEmail(value)) {
    return callback(new Error("邮箱格式错误"));
  }
  callback();
};
const validateMobile = (rule: any, value: string, callback: (e?: Error) => any): any => {
    if (!/\S/.test(value)) {
    return callback(new Error("手机号不能为空"));
  }
  if (value && !isMobile(value)) {
    return callback(new Error("手机格式错误"));
  }
  callback();
};

const validateIdCard = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (value && !isIdCard(value)) {
    return callback(new Error("身份证号格式错误"));
  }
  callback();
};
const rules = ref({
  email: [{ required:true, validator: validateEmail, trigger: "blur" }],
  mobile: [{  required: true,validator: validateMobile, trigger: "blur" }],
  password: [{ required: true,validator: validatePassword, trigger: "blur" }],
  confirmPassword: [{ required: true,validator: validateConfirmPassword, trigger: "blur" }],
  username: [{ required: true, message: "账号不能为空", trigger: "blur" }],
  realName: [{ required: true, message: "姓名不能为空", trigger: "blur" }],
  idCard: [{ required: true, validator:validateIdCard, trigger: "blur" }],
  gender: [{ required: true, message: "性别不能为空", trigger: "blur" }],
  headUrl: [{ required: true, message: "请上传头像图片", trigger: "blur" }],
});
const regSubmit = () => {
  regFormRef.value.validate((valid: boolean) => {
    if (valid) {
      baseService
        .post("/register", regForm)
        .then((res) => {
          if (res.code === 0) {
            ElMessage.success("注册成功");
            router.push("/login");
          } else {
            ElMessage.error(res.msg);
          }
        })
        .catch(() => {
        });
    }
  });
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.reg-container {
  height: 100vh; /* 改为视口高度 */
  width: 100%;
  overflow-y: auto; /* 改为auto只在需要时显示滚动条 */
  overflow-x: hidden;
  
  .reg-title {
    width: 100%;
    height: 185px;
    font-size: 24px;
    text-align: center;
    color: #ffffff;
    background-color: #2d3a4b;
  }

  .reg-titile-text {
    position: relative;
    top: 110px;
    width: 510px;
    height: 28px;
    margin: 0 auto;
  }

  .line {
    display: inline-block;
    width: 105px;
    height: 7px;
    border-top: 1px solid #fff;
  }

  .regForm {
    position: relative;
    top: -38px;
    width: 650px;
    background-color: #f2f2f2;
    margin: 15px auto;
    border-radius: 4px;
    min-height: 500px; /* 确保有足够高度 */
  }

  .regForm-text {
    min-height: 450px; /* 改为min-height */
    padding-bottom: 20px; /* 添加底部内边距 */
  }

  .textinfo1 {
    margin-left: 70px;
    margin-top: 40px;
  }

  .textinfo {
    margin-left: 70px;
    margin-top: 12px;
  }

  .regBtn {
    text-align: center;
    width: 650px;
    margin: 15px auto;
  }

  .reg {
    width: 25%;
    border-radius: 8px;
  }

  .reg-yzm {
    width: 40%;
    height: 39px;
    border-radius: 8px;
  }
}
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 100px;
  height: 50px;
  text-align: center;
}
.avatar {
  width: 50px;
  height: 50px;
  display: block;
}
</style>
