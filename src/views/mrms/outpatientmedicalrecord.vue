<template>
  <div class="mod-demo__mrmsoutpatientmedicalrecord">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item prop="examDate">
        <el-date-picker
         v-model="state.dataForm.examDate"
        type="date"
        placeholder="请选择预约时间"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker>  
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.patientName" placeholder="患者姓名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item> 
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
              <el-table-column prop="patientName" label="患者姓名" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="doctorName" label="医生姓名" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="examDate" label="预约时间" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="mainSuit" label="主诉" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="presentIll" label="现病史" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="bodyInfo" label="体格检查" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="diagnosisInfo" label="初步诊断" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="suggestion" label="处理意见" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="medicineInfo" label="药物信息" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="examFile" label="病历单" header-align="center" align="center" width="100">
                 <template v-slot="scope">
                  <a :href="fileUrl+scope.row.examFile" style="color:blue;cursor:pointer">{{ scope.row.examFile}}</a>
                </template>
              </el-table-column>
              <el-table-column prop="createDate" label="创建时间" header-align="center" align="center" width="100"></el-table-column>
              <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center" width="100"></el-table-column>
            <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button type="primary" link @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="danger" link @click="state.deleteHandle(scope.row.id)">删除</el-button>
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
import { reactive, ref, toRefs ,inject} from "vue";
import AddOrUpdate from "./outpatientmedicalrecord-add-or-update.vue";
import { useAppStore } from "@/store";
const fileUrl = inject('httpUrl')+"/recordUrl/"; // 注入全局变量

const store = useAppStore();
const view = reactive({
  getDataListURL: "/mrms/outpatientMedicalRecord/page",
  getDataListIsPage: true,
  deleteURL: "/mrms/outpatientMedicalRecord",  
  dataForm: {
    patientName: "",
    examDate:null
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
};
</script>
