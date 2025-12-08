<template>
  <div class="mod-demo__mrmsreservation">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item prop="reservationTime">
        <el-date-picker
         v-model="state.dataForm.reservationTime"
        type="date"
        placeholder="请选择预约时间"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker>  
      </el-form-item>
      <el-form-item prop="patientName">
        <el-input v-model="state.dataForm.patientName" placeholder="患者姓名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="patientName" label="患者姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="doctorName" label="医生姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="reservationTime" label="预约时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="status" label="预约状态" header-align="center" align="center">
        <template v-slot="scope">
          <el-tag v-if="scope.row.status===0" type="primary">已预约</el-tag>
          <el-tag v-if="scope.row.status===1" type="warning">取消预约</el-tag>
          <el-tag v-if="scope.row.status===2" type="success">已登记</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button type="primary" :disabled="scope.row.status!==0" link @click="addInPatientHandle(scope.row)">住院</el-button>
          <el-button type="success" :disabled="scope.row.status!==0" link @click="addOutPatientHandle(scope.row)">门诊登记</el-button>
          <el-button type="warning" :disabled="scope.row.status!==0" link @click="addOrUpdateHandle(scope.row.id)" >修改</el-button>
          <el-button type="danger" :disabled="scope.row.status!==1" link @click="state.deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
    <add-in-patient ref="addInPatientRef" @refreshDataList="state.getDataList">确定</add-in-patient>
    <add-out-patient ref="addOutPatientRef" @refreshDataList="state.getDataList">确定</add-out-patient>

  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { reactive, ref, toRefs } from "vue";
import AddOrUpdate from "./reservation-add-or-update.vue";
import addInPatient from "./add-in-patient.vue";
import addOutPatient from "./add-out-patient.vue";
import { IObject } from "@/types/interface";

import { useAppStore } from "@/store";
const store = useAppStore();

const view = reactive({
  getDataListURL: "/mrms/reservation/page",
  getDataListIsPage: true,
  deleteURL: "/mrms/reservation",  
  dataForm: {
    patientName: "",
    reservationTime:null
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
};
const addInPatientRef = ref();
const addInPatientHandle = (row?: IObject) => {
  addInPatientRef.value.init(row);
};
const addOutPatientRef = ref();
const addOutPatientHandle = (row?: IObject) => {
  addOutPatientRef.value.init(row);
};

</script>
