<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item label="科室名称" prop="officeName">
        <el-input v-model="dataForm.officeName" placeholder="科室名称"></el-input>
      </el-form-item>
      <el-form-item class="labelm" label="科室图片" prop="officePic">
          <el-upload
            class="avatar-uploader"
            :action="officePicUrl"
            :show-file-list="false"
            :on-success="successHandle"
            :before-upload="beforeUploadHandle"
          >
          <img v-if="dataForm.officePic" :src="officePicSrc" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
          </el-upload>
          <div slot="tip" class="el-upload__tip">
            只能上传jpg/png/gif文件，且不超过2M
          </div> 
      </el-form-item>
      <el-form-item label="科室位置" prop="officeAddr">
        <el-input v-model="dataForm.officeAddr" placeholder="科室位置"></el-input>
      </el-form-item> 
      </el-form>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref ,inject} from "vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";
const emit = defineEmits(["refreshDataList"]);
import app from "@/constants/app";
import { getToken } from "@/utils/cache";
import type { UploadProps } from 'element-plus'
const visible = ref(false);
const dataFormRef = ref();
const officePicUrl = ref("");
const officePicSrc = ref("");
const httpUrl = inject('httpUrl'); // 注入全局变量

const successHandle: UploadProps['onSuccess'] = (
  response,
  uploadFile
) => {
  if (response.code !== 0) {
    return ElMessage.error(response.msg);
  }
  officePicSrc.value = URL.createObjectURL(uploadFile.raw!);
  dataForm.officePic="/officeUrl/"+uploadFile.name;
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
 
const dataForm = reactive({
    id: '',  
    officeName: '',  
    officePicSrc: '',  
    officePic: '',  
    officeAddr: '',  
    delFlag: '',
  });

const rules = ref({
          officeName: [
      { required: true, message: '科室名称不能为空', trigger: 'blur' }
    ],
          officePic: [
      { required: true, message: '科室图片不能为空', trigger: 'blur' }
    ],
          officeAddr: [
      { required: true, message: '科室位置不能为空', trigger: 'blur' }
    ]
  });

const init = (id?: number) => {
  officePicUrl.value = `${app.api}/mrms/file/upload?token=${getToken()}&fileType=office`;
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
const getInfo = (id: number) => {
  baseService.get("/sys/office/" + id).then((res) => {
    Object.assign(dataForm, res.data);
    officePicSrc.value=httpUrl+dataForm.officePic;
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/office", {...dataForm}).then((res) => {
      ElMessage.success({
        message: '保存成功',
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
<style>
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
  width: 50px;
  height: 50px;
  text-align: center;
}
.avatar {
  width: 50px;
  height: 50px;
  display: block;
}
</style>