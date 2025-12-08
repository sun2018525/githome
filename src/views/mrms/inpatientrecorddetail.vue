<template>
  <el-dialog v-model="visible" title="住院登记记录" :close-on-click-modal="false" :close-on-press-escape="false" fullscreen>
  <div class="mod-demo__mrmsinpatientrecorddetail">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()" v-if="store.state.user.userType!==0">
      <el-form-item prop="inHospitalTime">
        <el-date-picker
         v-model="state.dataForm.recordTime"
        type="date"
        placeholder="请选择住院记录时间"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker> 
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item> 
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle(0,paramRow)">新增</el-button>
      </el-form-item> 
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border style="width: 100%">
      <el-table-column prop="recordTime" label="住院记录时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="patientName" label="患者姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="doctorName" label="医生姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="bodyChange" label="病情变化" header-align="center" align="center"></el-table-column>
      <el-table-column prop="treatment" label="诊疗措施" header-align="center" align="center"></el-table-column>
      <el-table-column prop="suggestion" label="查房意见" header-align="center" align="center"></el-table-column>
      <el-table-column prop="analysis" label="检查结果分析" header-align="center" align="center"></el-table-column>
      <el-table-column prop="drugInfo" label="药物信息" header-align="center" align="center"></el-table-column>
      <el-table-column prop="examFile" label="检查单" header-align="center" align="center">
        <template v-slot="scope">
          <a :href="fileUrl+scope.row.examFile" style="color:blue;cursor:pointer">{{ scope.row.examFile}}</a>
        </template>
      </el-table-column>
      <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column>
      {{ store.state.user.userType }}<el-table-column label="操作" fixed="right" header-align="center" align="center" width="150" v-if="store.state.user.userType!==0">
      <template v-slot="scope">
        <el-button type="primary"  link @click="addOrUpdateHandle(scope.row.id,paramRow)">修改</el-button>
        <el-button type="danger" link @click="state.deleteHandle(scope.row.id)">删除</el-button>
      </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
  </div>
  </el-dialog>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { reactive, ref, toRefs ,inject} from "vue";
import { IObject } from "@/types/interface";
import AddOrUpdate from "./inpatientrecorddetail-add-or-update.vue"; 
import { useAppStore } from "@/store";
const fileUrl = inject('httpUrl')+"/recordUrl/"; // 注入全局变量
const store = useAppStore();

const view = reactive({
  getDataListURL: "/mrms/inpatientrecordDetail/list",
  deleteURL: "/mrms/inpatientrecordDetail",
  dataForm:{
    recordTime:null
  }
});
const paramRow={
    patientId: '',  
    patientName: '',  
    doctorId: '',  
    doctorName: '',  
    inpatientId: ''
}
const visible = ref(false);
const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number,row?:IObject) => {
  addOrUpdateRef.value.init(id,row);
};
const init = (row?: IObject) => {
  visible.value = true;
  Object.assign(paramRow, row);
};
defineExpose({
  init
});
</script>
