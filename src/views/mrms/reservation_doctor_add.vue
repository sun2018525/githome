<template>
  <el-dialog v-model="visible" title="预约" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" ref="dataFormRef" :rules="rules" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item prop="reservationTime" label="预约时间" >
         <el-date-picker
         v-model="dataForm.reservationTime"
        type="date"
        placeholder="请选择预约时间"
        :disabledDate="disabledDate"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker>  
      </el-form-item> 
    </el-form>
    <template v-slot:footer>
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
  doctorId: null,
  doctorName:"",
  reservationTime:null,
  officeName:""
});
const rules = ref({
    reservationTime: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ] 
  });  
const init = (doctor:IObject) => {
  visible.value = true; 
  dataForm.doctorId=doctor.id;
  dataForm.doctorName=doctor.realName;
  dataForm.officeName=doctor.officeName;
  dataForm.reservationTime=null;
};
const disabledDate = (time: Date) => {
  const fifteenDays = 14 * 24 * 3600 * 1000; // 14天的时间戳
  return time.getTime() > (Date.now() + fifteenDays)|| (time.getTime() < Date.now())
  
} 
// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    baseService.post("/mrms/reservation/reservationSave", {
      ...dataForm
    }).then((res) => {
      ElMessage.success({
        message: "预约成功，请至【我的预约】界面查看预约信息！",
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

<style lang="less" scoped>
.mod-sys__user {
  .role-list {
    .el-select {
      width: 100%;
    }
  }
}
</style>
