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
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item> 
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="patientName" label="患者姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="doctorName" label="医生姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="officeName" label="科室名称" header-align="center" align="center"></el-table-column>
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
          <el-button type="warning" :disabled="scope.row.status!==0" link @click="cancel(scope.row.id)">取消预约</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { reactive, toRefs } from "vue";
import AddOrUpdate from "./reservation-add-or-update.vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/mrms/reservation/page",
  getDataListIsPage: true,
  deleteURL: "/mrms/reservation",  
  dataForm: {
    patientName:null,
    reservationTime: null
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const cancel=(id:number)=>{
  state.dataListLoading=true;
  baseService.put("/mrms/reservation/cancel/"+id).then((res) => {
    ElMessage.success({
      message: '取消预约成功',
      duration: 500,
      onClose: () => {
        state.dataListLoading = false;
        state.getDataList();
      }
    });
  });
}
</script>