<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item prop="username" label="账号">
        <el-input v-model="dataForm.username" placeholder="账号"></el-input>
      </el-form-item>
      <el-form-item prop="realName" label="真实姓名">
        <el-input v-model="dataForm.realName" placeholder="真实姓名"></el-input>
      </el-form-item>
      <el-form-item prop="password" label="密码" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.password" type="password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item prop="confirmPassword" label="确认密码" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.confirmPassword" type="password" placeholder="确认密码"></el-input>
      </el-form-item>
      <el-form-item prop="idCard" label="身份证号">
        <el-input v-model="dataForm.idCard" placeholder="请输入15位或18位身份证号码"></el-input>
      </el-form-item>
      <el-form-item prop="gender" label="性别">
        <ren-radio-group v-model="dataForm.gender" dict-type="gender"></ren-radio-group>
      </el-form-item>
      <el-form-item prop="email" label="邮箱">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <el-form-item prop="mobile" label="手机号">
        <el-input v-model="dataForm.mobile" placeholder="手机号"></el-input>
      </el-form-item> 
            <el-form-item prop="address" label="住址">
        <el-input v-model="dataForm.address" placeholder="住址"></el-input>
      </el-form-item>
      <el-form-item class="labelm" label="头像" prop="headUrl">
          <el-upload
            class="avatar-uploader"
            :action="headPicUrl"
            :show-file-list="false"
            :on-success="successHandle"
            :before-upload="beforeUploadHandle"
          >
          <img v-if="dataForm.headUrl" :src="headPicSrc" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
          </el-upload>
          <div slot="tip" class="el-upload__tip">
            只能上传jpg/png/gif文件，且不超过2M
          </div> 
      </el-form-item>
      <el-form-item prop="status" label="状态">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">停用</el-radio>
          <el-radio :label="1">正常</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref, inject } from "vue";
import baseService from "@/service/baseService";
import { isEmail, isMobile ,isIdCard} from "@/utils/utils";
import { ElMessage } from "element-plus";
import app from "@/constants/app";
import { getToken } from "@/utils/cache";
import type { UploadProps } from 'element-plus'
const httpUrl = inject('httpUrl'); // 注入全局变量

const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();
const headPicUrl = ref(`${app.api}/mrms/file/upload?token=${getToken()}&fileType=head`);//上传的地址
const headPicSrc = ref("");//显示头像

const dataForm = reactive({
  id: "",
  username: "",
  idCard: "", 
  password: "",
  confirmPassword: "",
  realName: "",
  gender: 0,
  email: "",
  mobile: "", 
  address:"",
  status: 1,
  userType:0,
  superAdmin:0,
  headUrl:""
});
const successHandle: UploadProps['onSuccess'] = (
  response,
  uploadFile
) => {
  if (response.code !== 0) {
    return ElMessage.error(response.msg);
  }
  headPicSrc.value = URL.createObjectURL(uploadFile.raw!);
  dataForm.headUrl="/headUrl/"+uploadFile.name;
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
const validatePassword = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (!dataForm.id && !/\S/.test(value)) {
    return callback(new Error("密码不能为空"));
  }
  callback();
};
const validateConfirmPassword = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (!dataForm.id && !/\S/.test(value)) {
    return callback(new Error("确认密码不能为空"));
  }
  if (dataForm.password !== value) {
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
  username: [{ required: true, message: "必填项不能为空", trigger: "blur" }],
  password: [{ required: true,  validator: validatePassword, trigger: "blur" }],
  confirmPassword: [{ required: true, validator: validateConfirmPassword, trigger: "blur" }],
  idCard: [{ required: true, validator: validateIdCard, trigger: "blur" }],
  realName: [{ required: true, message: "必填项不能为空", trigger: "blur" }],
  email: [{ required: true, validator: validateEmail, trigger: "blur" }],
  mobile: [{ required: true, validator: validateMobile, trigger: "blur" }],
  headUrl: [{ required: true, message: "请上传头像", trigger: "blur" }],
  gender: [{required: true,  message: "必填项不能为空", trigger: "blur" }],

});
const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }
  if (id) {
    getInfo(id);
  }
};
 
// 获取信息
const getInfo = (mrmsId: number) => {
  baseService.get(`/sys/user/mrmsUser/`+mrmsId).then((res) => {
    Object.assign(dataForm, res.data);

    dataForm.confirmPassword=dataForm.password;
    headPicSrc.value=httpUrl+dataForm.headUrl;
  });
};
// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/user", {
      ...dataForm
    }).then((res) => {
      ElMessage.success({
        message: "保存成功",
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};

defineExpose({
  init
});
</script>

<style lang="less" scoped>
.mod-sys__user {
  .role-list {
    .el-select {
      width: 100%;
    }
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
