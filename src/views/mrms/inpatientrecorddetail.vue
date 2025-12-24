<template>
  <el-dialog v-model="visible" title="住院登记记录" :close-on-click-modal="false" :close-on-press-escape="false" fullscreen>
  <div class="mod-demo__mrmsinpatientrecorddetail">
    <el-form :inline="true" :model="dataForm" @keyup.enter="getDataList(dataForm.inpatientId)" v-if="store.state.user.userType!==0">
      <el-form-item prop="inpatientId" v-show="false">
        <el-input v-model="dataForm.inpatientId" ></el-input> 
      </el-form-item> 
       <el-form-item prop="recordTime">
        <el-date-picker
         v-model="dataForm.recordTime"
        type="date"
        placeholder="请选择住院记录时间"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"></el-date-picker>
        </el-form-item>
      <el-form-item>
        <el-button @click="getDataList(dataForm.inpatientId)">查询</el-button>
      </el-form-item> 
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle(0,paramRow)">新增</el-button>
      </el-form-item> 
    </el-form>
    <el-table v-loading="dataListLoading" :data="dataList" border style="width: 100%">
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
        <el-button type="danger" link @click="deleteHandle(scope.row.id)">删除</el-button>
      </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="getDataList(dataForm.inpatientId)">确定</add-or-update>
  </div>
  </el-dialog>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { ElMessageBox,ElMessage} from "element-plus";
import { reactive, ref,toRefs, watch ,inject} from "vue";
import baseService from "@/service/baseService";
import { IObject } from "@/types/interface";
import AddOrUpdate from "./inpatientrecorddetail-add-or-update.vue"; 
import { useAppStore } from "@/store";
const fileUrl = inject('httpUrl')+"/recordUrl/"; // 注入全局变量
const store = useAppStore();
const props = defineProps(['inpatientId']); 
// const view = reactive({
  // getDataListURL: "/mrms/inpatientrecordDetail/list",
  // deleteURL: "/mrms/inpatientrecordDetail",
  // dataForm:{
  //   recordTime:null,
  //   inpatientId:null
  // }
// });
const dataList= ref([]);
const dataListLoading=ref(false);
// 将 dataForm 定义为响应式对象
const dataForm = reactive({
  recordTime: null,
  inpatientId: 0
});
const paramRow={
    patientId: '',  
    patientName: '',  
    doctorId: '',  
    doctorName: ''
};
const visible = ref(false);
// const state = reactive({ ...useView(view), ...toRefs(view) });
// watch(() => props.inpatientId, async (newId: any) => {
//   alert()
//   if (newId) {
//     await getDataList(newId); // 根据ID获取详细信息
//   } else {
//     dataList.value = [];
//   }
// }, { immediate: true }); // 立即执行一次，以处理初始加载情况

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number,row?:IObject) => {
  addOrUpdateRef.value.init(id,row);
};
const init = (row?: IObject) => {
  dataForm.inpatientId=row?.id;
  visible.value = true;
  Object.assign(paramRow, row);
  dataForm.recordTime=null;
  getDataList(dataForm.inpatientId);
};
// 获取信息
const getDataList = (id: number) => {
  dataForm.inpatientId=id;
  baseService.get("/mrms/inpatientrecordDetail/list",dataForm).then((res) => {
    dataListLoading.value = false;
    dataList.value = res.data;
  });
}; 
const deleteHandle = (id: any) => {
   ElMessageBox.confirm("确定进行[删除]操作?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
          baseService.delete("/mrms/inpatientrecordDetail/"+id,id).then((res) => {
          dataListLoading.value = false; 
            ElMessage.success({
            message: "删除成功",
            duration: 500,
            onClose: () => {
              getDataList(dataForm.inpatientId);
            }
          });
  }).catch(()=>{
 ElMessage.error('删除失败');
  }); 
}).catch(()=>{
      ElMessage.info('已取消删除');
})
}; 
defineExpose({
  init
});
</script>
