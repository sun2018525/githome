<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="140px">
      <el-form-item label="入院时间" prop="inHospitalTime">
        <el-date-picker
          v-model="dataForm.inHospitalTime"
          type="date"
          placeholder="请选择入院时间"
          format="YYYY-MM-DD"
          value-format="YYYY-MM-DD">
        </el-date-picker>        
      </el-form-item>
      <el-form-item label="出院时间" prop="outHospitalTime" >
          <el-date-picker
          v-model="dataForm.outHospitalTime"
          type="date"
          placeholder="请选择出院时间"
          format="YYYY-MM-DD"
          value-format="YYYY-MM-DD"
          disabled>
        </el-date-picker>              
      </el-form-item>
        <el-form-item label="住院状态" prop="inHospitalStatus">
         <el-select v-model="dataForm.inHospitalStatus" placeholder="住院状态" disabled clearable>
          <el-option label="入院" :value="0"></el-option>
          <el-option label="出院" :value="1"></el-option>
        </el-select>      
      </el-form-item>
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
          <el-form-item label="全面体格检查结果" prop="bodyInfo">
        <el-input v-model="dataForm.bodyInfo" placeholder="全面体格检查结果"></el-input>
      </el-form-item>
          <el-form-item label="出院诊断" prop="outoutDiacrisis" >
        <el-input v-model="dataForm.outDiacrisis" placeholder="出院诊断" disabled></el-input>
      </el-form-item>
          <el-form-item label="出院医嘱" prop="outAdvice">
        <el-input v-model="dataForm.outAdvice" placeholder="出院医嘱" disabled></el-input>
      </el-form-item>
          <el-form-item label="出院用药" prop="outDrug">
        <el-input v-model="dataForm.outDrug" placeholder="出院用药" disabled></el-input>
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
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: null,  
  reservationId: null, 
  medicalNum: '',  
  patientId: null,  
  patientName: '',  
  doctorId: null,  
  doctorName: '',  
  inHospitalTime: '',  
  outHospitalTime: '',  
  inHospitalStatus: 0,  
  inHospitalReason: '',  
  bedInfo: '',  
  process: '',  
  presentHistory: '',  
  pastHistory: '',  
  familyHistory: '',  
  bodyInfo: '',  
  outDiacrisis: '',  
  outAdvice: '',  
  outDrug: ''
}); 

const rules = ref({
    inHospitalTime: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
    //       outHospitalTime: [
    //   { required: true, message: '必填项不能为空', trigger: 'blur' }
    // ],
          inHospitalStatus: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
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
    ],
    //       outDiacrisis: [
    //   { required: true, message: '必填项不能为空', trigger: 'blur' }
    // ],
    //       outAdvice: [
    //   { required: true, message: '必填项不能为空', trigger: 'blur' }
    // ],
    //       outDrug: [
    //   { required: true, message: '必填项不能为空', trigger: 'blur' }
    // ]
});
const init = (id?: number) => {
  visible.value = true;
  dataForm.id = null;
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
    (!dataForm.id ? baseService.post : baseService.put)("/mrms/inpatientRecord", dataForm).then((res) => {
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
