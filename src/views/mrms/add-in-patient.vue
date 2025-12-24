<template>
  <el-dialog v-model="visible" title="住院登记" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="140px">
      <el-form-item label="入院情况" prop="inHospitalReason">
        <el-input v-model="dataForm.inHospitalReason" placeholder="入院情况"></el-input>
      </el-form-item>
      <el-form-item label="床位信息" prop="bedInfo">
        <el-input v-model="dataForm.bedInfo" placeholder="床位信息"></el-input>
      </el-form-item>
      <el-form-item label="诊疗经过" prop="process">
        <el-input v-model="dataForm.process" placeholder="诊疗经过"></el-input>
      </el-form-item>
      <el-form-item label="现病史" prop="presentHistory">
        <el-input v-model="dataForm.presentHistory" placeholder="现病史"></el-input>
      </el-form-item>                    
      <el-form-item label="既往史" prop="pastHistory">
        <el-input v-model="dataForm.pastHistory" placeholder="既往史"></el-input>
      </el-form-item>
      <el-form-item label="家族史" prop="familyHistory">
        <el-input v-model="dataForm.familyHistory" placeholder="家族史"></el-input>
      </el-form-item>
      <el-form-item label="全面体格检查结果" prop="bodyInfo" >
        <el-input v-model="dataForm.bodyInfo" placeholder="全面体格检查结果"></el-input>
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
import { ElMessage } from "element-plus";
import { IObject } from "@/types/interface";

const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: null,  
  reservationId:null,
  inHospitalReason: '',  
  bedInfo: '',  
  process: '', 
  presentHistory: '',  
  pastHistory:'',
  familyHistory:'',  
  bodyInfo: ''  
});

const rules = ref({
      
          inHospitalReason: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          bedInfo: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          process: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ], 
          presentHistory: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          pastHistory: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ], 
       familyHistory: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          bodyInfo: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ]
  });

const init = (row:IObject) => {
  visible.value = true;
   // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }
  Object.assign(dataForm, row);
  dataForm.reservationId=row.id;
  dataForm.id = null;
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/mrms/inpatientRecord/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    baseService.post("/mrms/inpatientRecord", dataForm).then((res) => {
      ElMessage.success({
        message: '保存成功，请至【住院管理】界面查看',
        duration: 1500,
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
