<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
          
          <el-form-item label="病情变化" prop="bodyChange">
        <el-input v-model="dataForm.bodyChange" placeholder="病情变化"></el-input>
      </el-form-item>
          <el-form-item label="诊疗措施" prop="treatment">
        <el-input v-model="dataForm.treatment" placeholder="诊疗措施"></el-input>
      </el-form-item>
          <el-form-item label="查房意见" prop="suggestion">
        <el-input v-model="dataForm.suggestion" placeholder="查房意见"></el-input>
      </el-form-item>
          <el-form-item label="检查结果分析" prop="analysis">
        <el-input v-model="dataForm.analysis" placeholder="检查结果分析"></el-input>
      </el-form-item>
          <el-form-item label="药物信息" prop="drugInfo">
        <el-input v-model="dataForm.drugInfo" placeholder="药物信息"></el-input>
      </el-form-item>
      <el-form-item label="检查单" prop="examFile">
        <el-upload ref="uploadRef" v-model:file-list="fileList" :on-change="handleChange" :action="recordUrl" :before-upload="beforeUploadHandle" :on-success="successHandle" class="upload-demo">
           <el-button type="primary">点击上传</el-button>
          <template #tip>
            <!-- <span v-if="dataForm.examFile">{{ dataForm.examFile}}</span> -->
            <div class="el-upload__tip">
              只能上传一个文件，且文件大小不能超过10M！
            </div>
          </template>
          <!-- <span v-if="dataForm.examFile"><a :href="fileUrl+dataForm.examFile" style="color:blue;cursor:pointer">{{ dataForm.examFile}}</a></span>
          <el-icon v-else class="el-icon--upload"><upload-filled  /></el-icon>
          <div class="el-upload__text">将文件拖到此处，或点击上传</div> -->
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
import { reactive, ref ,inject,nextTick} from "vue";
import baseService from "@/service/baseService";
import { ElMessage,UploadProps,UploadInstance ,UploadUserFile} from "element-plus";
import { IObject } from "@/types/interface";
import app from "@/constants/app";
import { getToken } from "@/utils/cache";
const uploadRef = ref<UploadInstance>()
const emit = defineEmits(["refreshDataList"]);
const fileUrl = inject('httpUrl')+"/recordUrl/"; // 注入全局变量

const visible = ref(false);
const dataFormRef = ref();
const recordUrl = ref("");
const fileList = ref<UploadUserFile[]>([])

const dataForm = reactive({
  id: '',  
  patientId: '',  
  patientName: '',  
  doctorId: '',  
  doctorName: '',  
  inpatientId: '',  
  recordTime: '',  
  bodyChange: '',  
  treatment: '',  
  suggestion: '',  
  analysis: '',  
  drugInfo: '',  
  examFile: ''
});

const rules = ref({
         
          bodyChange: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          treatment: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          suggestion: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          analysis: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          drugInfo: [
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
  dataForm.examFile=uploadFile.name;     
}

const beforeUploadHandle: UploadProps['beforeUpload'] = (file) => {
  if (file.size / 1024 / 1024 > 10) {
    ElMessage.error('文件大小不能超过10M!')
    return false
  }
  return true
}
const handleExceed: UploadProps['onExceed'] = (files) => {
   ElMessage.warning('只能上传一个文件，请先删除已上传文件');
}
const handleChange: UploadProps['onChange'] = (uploadFile, uploadFiles) => {
   if (uploadFiles.length > 1) {
    uploadFiles.splice(0, 1); // 自动替换前一个文件
  }
  // if(fileList.value.length>1){
      fileList.value=[{name: uploadFile.name, url: URL.createObjectURL(uploadFile.raw!)}];
  // }
};
const init = (id:number,row: IObject) => {
  recordUrl.value = `${app.api}/mrms/file/upload?token=${getToken()}&fileType=record`;
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  nextTick(() => {
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
    uploadRef.value!.clearFiles();
  }
});
 
  Object.assign(dataForm, row);  
  dataForm.inpatientId=row.id;
  dataForm.id = "";

  if (id) {
    getInfo(id);
  }
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/mrms/inpatientrecordDetail/" + id).then((res) => {
    Object.assign(dataForm, res.data);
    // const blob = new Blob([fileUrl+dataForm.examFile], { type: 'text/plain' })
    // const file = new File([blob], dataForm.examFile, { type: 'text/plain' });
    // fileList.value.push({name: dataForm.examFile, url: URL.createObjectURL(file)});
     fileList.value = [{
        name: dataForm.examFile,
        url: fileUrl + dataForm.examFile
      }]; 
  });
}; 

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/mrms/inpatientrecordDetail", dataForm).then((res) => {
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
