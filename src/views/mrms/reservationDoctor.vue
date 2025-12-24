
<template>
  <el-space style="width: 100%" fill>
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item prop="officeList">
        <el-select v-model="state.dataForm.officeId" placeholder="请选择科室" clearable>
          <el-option v-for="item in officeList" :key="item.id" :label="item.officeName" :value="item.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="setLoading">查询</el-button>
      </el-form-item>
    </el-form>
    <el-skeleton :loading="loading" animated :count="3">
      <template #template>
        <div style="flex: 1">
          <el-skeleton-item variant="image" style="height: 240px; width: 200px" />
          <div style="padding: 14px">
            <el-skeleton-item variant="h3" style="width: 50%" />
            <div style="
                display: flex;
                align-items: center;
                justify-items: space-between;
                margin-top: 16px;
                height: 16px;
              ">
              <el-skeleton-item variant="text" style="margin-right: 16px" />
              <el-skeleton-item variant="text" style="width: 30%" />
            </div>
          </div>
        </div>
      </template>
      <template #default>
        <el-card style="max-width: 200px; min-width: 200px;margin-left:20px;" v-for="item in state.dataList" :key="item.id"
          :body-style="{ padding: '0px', marginBottom: '1px' }" @click="reservationHandle(item)">
          <img :src="httpUrl + item.headUrl" class="image multi-content doctor-image"
            style="width: 200px; height: 270px; object-fit: cover" :alt="`${item.realName}医生照片`" />
          <div style="padding: 14px">
            <span style="font-weight: bold; font-size: 16px">{{ item.realName }}</span><br />
            <span style="font-size: 14px; color: #666">所属科室：{{ item.officeName }}</span>
            <div class="bottom card-header">
              <div class="time" style="font-size: 12px; color: #999; margin-top: 8px">{{ item.doctorInfo }}</div>
            </div>
          </div>
        </el-card>
      </template>
    </el-skeleton>
  </el-space>
  <reservation-doctor-add ref="reservationDoctorAddRef" @refreshDataList="state.getDataList">确定</reservation-doctor-add>
</template>

<script lang="ts" setup>
import { reactive, onMounted, ref, toRefs, inject } from 'vue'
import useView from "@/hooks/useView";
import baseService from "@/service/baseService";
import ReservationDoctorAdd from "./reservation_doctor_add.vue";
const httpUrl = inject('httpUrl'); // 注入全局变量

const officeList = ref<any[]>([]);
const view = reactive({
  getDataListURL: "/mrms/reservation/queryDoctorList",
  getDataListIsPage: false,
  deleteIsBatch: false,
  dataForm: {
    officeId: null
  }
});
// 获取科室列表
const getOfficeList = () => {
  return baseService.get("/sys/office/list").then((res) => {
    if (res.code === 0) {
      officeList.value = res.data;
    }
  });
};

const state = reactive({ ...useView(view), ...toRefs(view) });

const loading = ref(true)

const setLoading = () => {
  loading.value = true
  setTimeout(() => {
    loading.value = false
    state.getDataList();
  }, 1000)
}
onMounted(() => {
  getOfficeList();
  loading.value = false
})
const reservationDoctorAddRef = ref();
const reservationHandle = (doctor?: object) => {
  reservationDoctorAddRef.value.init(doctor);
};
</script>
<style scoped> 
.el-skeleton {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  justify-content: flex-start;
}

.el-space {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
</style>