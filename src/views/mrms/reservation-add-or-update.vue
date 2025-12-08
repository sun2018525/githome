<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item label="患者姓名" prop="patientId">
        <el-select v-model="dataForm.patientId"  placeholder="请选择患者">
          <el-option
            v-for="item in patientList"
            :key="item.id"
            :label="item.realName"
            :value="item.id"
          ></el-option>
        </el-select>
       </el-form-item>
      <el-form-item label="医生姓名" prop="doctorId">
        <el-select v-model="dataForm.doctorId"  placeholder="请选择医生">
          <el-option
            v-for="item in doctorList"
            :key="item.id"
            :label="item.realName"
            :value="item.id"
          ></el-option>
        </el-select>
       </el-form-item>
       <el-form-item label="预约时间" prop="reservationTime">
          <el-date-picker
          v-model="dataForm.reservationTime"
          type="date"
          placeholder="请选择预约时间"
          :disabledDate="disabledDate"
          format="YYYY-MM-DD"
          value-format="YYYY-MM-DD"></el-date-picker>  
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
const patientList = ref<any[]>([]);
const doctorList = ref<any[]>([]);

const dataForm = reactive({
  id: null,
  patientId: null, 
  doctorId: null, 
  reservationTime: null,
  status:0
});

const rules = ref({
    patientId: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
    doctorId: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
    reservationTime: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ] 
  });
// 获取患者列表
const getPatientList = () => {
  return baseService.get("/sys/user/patientList").then((res) => {
    if(res.code===0){
      patientList.value = res.data;
    }
  });
};
// 获取医生列表
const getDoctorList = () => {
  return baseService.get("/sys/user/doctorList").then((res) => {
    if(res.code===0){
      doctorList.value = res.data;
    }
  });
};
const disabledDate = (time: Date) => {
  const fifteenDays = 14 * 24 * 3600 * 1000; // 15天的时间戳
  return time.getTime() > (Date.now() + fifteenDays)|| (time.getTime() < Date.now())
} 
const init = (id?: number) => {
  visible.value = true;
  dataForm.id = null;
  getPatientList();
  getDoctorList();
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
  baseService.get("/mrms/reservation/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/mrms/reservation", dataForm).then((res) => {
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
