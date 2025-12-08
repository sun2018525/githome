<template>
  <div class="mod-demo__mrmsinpatientrecord">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item prop="medicalNum">
         <el-input v-model="state.dataForm.medicalNum" placeholder="病案号" clearable></el-input>
      </el-form-item>
      <el-form-item prop="inHospitalTime">
        <el-date-picker
         v-model="state.dataForm.inHospitalTime"
        type="date"
        placeholder="请选择入院时间"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker> 
      </el-form-item>
      <el-form-item prop="outHospitalTime">
        <el-date-picker
         v-model="state.dataForm.outHospitalTime"
        type="date"
        placeholder="请选择出院时间"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker> 
      </el-form-item>
      <el-form-item prop="inHospitalStatus">
         <el-select v-model="state.dataForm.inHospitalStatus" placeholder="住院状态" clearable>
         <el-option label="入院" :value="0"></el-option>
         <el-option label="出院" :value="1"></el-option>
        </el-select>
      </el-form-item> 
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
          <el-table-column prop="medicalNum" label="病案号" header-align="center" align="center"></el-table-column>
          <el-table-column prop="patientName" label="患者姓名" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="doctorName" label="医生姓名" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="inHospitalTime" label="入院时间" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="outHospitalTime" label="出院时间" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="inHospitalStatus" label="住院状态" header-align="center" align="center" width="100">
            <template v-slot="scope">
              <el-tag v-if="scope.row.inHospitalStatus===0" type="primary">入院</el-tag>
              <el-tag v-if="scope.row.inHospitalStatus===1" type="success">出院</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="inHospitalReason" label="入院情况" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="bedInfo" label="床位信息" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="process" label="诊疗经过" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="presentHistory" label="现病史" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="pastHistory" label="既往史" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="familyHistory" label="家族史" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="bodyInfo" label="全面体格检查结果" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="outDiacrisis" label="出院诊断" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="outAdvice" label="出院医嘱" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="outDrug" label="出院用药" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="createDate" label="创建时间" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center" width="100"></el-table-column>
          <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
          <template v-slot="scope">
            <el-button type="info" link @click="addRecordDetailHandle(scope.row)">病历登记详情</el-button>
            <el-button type="primary" :disabled="scope.row.inHospitalStatus===1" link @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
            <el-button type="danger" :disabled="scope.row.inHospitalStatus===1" link @click="state.deleteHandle(scope.row.id)">删除</el-button>
            <el-button type="success" :disabled="scope.row.inHospitalStatus===1" link @click="updateOutPatientHandle(scope.row)">出院</el-button>
          </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
    <add-record-detail ref="addRecordDetailRef"></add-record-detail>
    <update-out-patient ref="updateOutPatientRef" @refreshDataList="state.getDataList">确定</update-out-patient>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { reactive, ref, toRefs } from "vue";
import AddOrUpdate from "./inpatientrecord-add-or-update.vue";
import updateOutPatient from "./update-out-patient.vue";
import AddRecordDetail from "./inpatientrecorddetail.vue";
import { ElMessage, ElMessageBox } from "element-plus";
import baseService from "@/service/baseService";
import { IObject } from "@/types/interface";

const view = reactive({
  getDataListURL: "/mrms/inpatientRecord/page",
  getDataListIsPage: true,
  deleteURL: "/mrms/inpatientRecord",  
  dataForm: {
    patientName: '',
    inHospitalTime:null,
    inHospitalStatus:null,
    outHospitalTime:null,
    medicalNum:""
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
};
const addRecordDetailRef = ref();
const addRecordDetailHandle = (row?: IObject) => {
  addRecordDetailRef.value.init(row);
};
const updateOutPatientRef= ref();
const updateOutPatientHandle = (row?: IObject) => {
  updateOutPatientRef.value.init(row);
};
const updateInPatientStatus = (id:number) => {
  ElMessageBox.confirm("确定进行[出院]操作?", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
    })
    .then(() => {
      state.dataListLoading = true;
      baseService.put("/mrms/inpatientRecord/updateInPatientStatus",{id:id,inHospitalStatus:'1'}).then((res) => {
      ElMessage.success({
        message: '提交成功',
        duration: 500,
        onClose: () => {
          state.dataListLoading = false;
          state.getDataList();
        }
      });
    });
  });
}
</script>
