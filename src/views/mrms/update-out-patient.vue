<template>
  <el-dialog v-model="visible" title="出院" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
          <el-form-item label="出院诊断" prop="outDiacrisis">
        <el-input v-model="dataForm.outDiacrisis" placeholder="出院诊断"></el-input>
      </el-form-item>
          <el-form-item label="出院医嘱" prop="outAdvice">
        <el-input v-model="dataForm.outAdvice" placeholder="出院医嘱"></el-input>
      </el-form-item>
          <el-form-item label="出院用药" prop="outDrug">
        <el-input v-model="dataForm.outDrug" placeholder="出院用药"></el-input>
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
  inHospitalStatus:1,
  outDiacrisis: '',  
  outAdvice: '',  
  outDrug: ''
});

const rules = ref({
          outDiacrisis: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          outAdvice: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          outDrug: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ]
  });

const init = (row?: IObject) => {
  visible.value = true; 
  dataForm.id = null;
  Object.assign(dataForm, row);
  dataForm.inHospitalStatus=1;
};
 
// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
  if (!valid) {
    return false;
  }
  baseService.put("/mrms/inpatientRecord/updateOutPatient",dataForm).then((res) => {
  ElMessage.success({
    message: '提交成功',
    duration: 500,
    onClose: () => {
      visible.value = false;
      emit("refreshDataList");
    }
  }); 
})
})
};

defineExpose({
  init
});
</script>
