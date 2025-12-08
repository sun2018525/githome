<template>
  <el-dialog v-model="visible" title="门诊病历登记" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
          <el-form-item label="主诉" prop="mainSuit">
        <el-input v-model="dataForm.mainSuit" placeholder="主诉"></el-input>
      </el-form-item>
          <el-form-item label="现病史" prop="presentIll">
        <el-input v-model="dataForm.presentIll" placeholder="现病史"></el-input>
      </el-form-item>
          <el-form-item label="体格检查" prop="bodyInfo">
        <el-input v-model="dataForm.bodyInfo" placeholder="体格检查"></el-input>
      </el-form-item>
          <el-form-item label="初步诊断" prop="diagnosisInfo">
        <el-input v-model="dataForm.diagnosisInfo" placeholder="初步诊断"></el-input>
      </el-form-item>
          <el-form-item label="处理意见" prop="suggestion">
        <el-input v-model="dataForm.suggestion" placeholder="处理意见"></el-input>
      </el-form-item>
          <el-form-item label="药物信息" prop="medicineInfo">
        <el-input v-model="dataForm.medicineInfo" placeholder="药物信息"></el-input>
      </el-form-item>
      <el-form-item label="病历单" prop="examFile">
        <el-upload :action="recordUrl" drag multiple :before-upload="beforeUploadHandle" :on-success="successHandle" class="text-center">
          <el-icon class="el-icon--upload"><upload-filled /></el-icon>
          <div class="el-upload__text">将文件拖到此处，或点击上传</div>
        </el-upload>
       </el-form-item>
      </el-form>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { ElMessage,UploadProps } from "element-plus";
import { IObject } from "@/types/interface";
import app from "@/constants/app";
import { getToken } from "@/utils/cache";
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();
const recordUrl = ref("");
const dataForm = reactive({
  id: null,   
  reservationId:null,
  mainSuit: '',  
  presentIll: '',  
  bodyInfo: '',  
  diagnosisInfo: '',  
  suggestion: '',  
  medicineInfo: '',  
  examFile: '',  
  examFileDisplay:''
});

const rules = ref({
         
          mainSuit: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          presentIll: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          bodyInfo: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          diagnosisInfo: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          suggestion: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          medicineInfo: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          examFile: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ] 
  });
const successHandle: UploadProps['onSuccess'] = (
  response,
  uploadFile
) => {
  if (response.code !== 0) {
    return ElMessage.error(response.msg);
  }
  dataForm.examFileDisplay = URL.createObjectURL(uploadFile.raw!);
  dataForm.examFile=uploadFile.name;
}

const beforeUploadHandle: UploadProps['beforeUpload'] = (file) => {
  if (file.size / 1024 / 1024 > 2) {
    ElMessage.error('文件大小不能超过2M!')
    return false
  }
  return true
}
const init = (row:IObject) => {
  visible.value = true;
  dataForm.id = null;
  recordUrl.value = `${app.api}/mrms/file/upload?token=${getToken()}&fileType=record`;
    // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }
  Object.assign(dataForm, row);
  dataForm.reservationId=row.id;
};


// 获取信息
const getInfo = (id: number) => {
  baseService.get("/mrms/outpatientMedicalRecord/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/mrms/outpatientMedicalRecord", dataForm).then((res) => {
      ElMessage.success({
        message: '保存成功，请至【门诊病历】界面查看',
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
