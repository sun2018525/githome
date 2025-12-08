<template>
  <div class="mod-sys__user">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.username" placeholder="账号" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <ren-select v-model="state.dataForm.gender" dict-type="gender" placeholder="性别"></ren-select>
      </el-form-item>
      <el-form-item>
        <el-select v-model="state.dataForm.officeId"  placeholder="请选择科室" clearable>
          <el-option
          v-for="item in officeList"
          :key="item.id"
          :label="item.officeName"
          :value="item.id"
        />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:user:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:user:delete')" type="danger" @click="state.deleteHandle()">批量删除</el-button>
      </el-form-item>
      <!-- <el-form-item>
        <el-button v-if="state.hasPermission('sys:user:export')" type="info" @click="state.exportHandle()">导出</el-button>
      </el-form-item> -->
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" @sort-change="state.dataListSortChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="headUrl" label="头像" header-align="center" align="center" width="100">
        <template #default="{ row }">
          <img
            v-if="row.headUrl"
            :src="httpUrl+row.headUrl"
            class="user-avatar"
          />
          <span v-else class="no-avatar">无头像</span>
        </template>
      </el-table-column>
      <el-table-column prop="username" label="账号" sortable="custom" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column prop="realName" label="姓名" sortable="custom" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column prop="officeName" label="科室名称" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column prop="mobile" label="手机号" sortable="custom" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column prop="gender" label="性别" sortable="custom" header-align="center" align="center" >
        <template v-slot="scope">
          {{ state.getDictLabel("gender", scope.row.gender) }}
        </template>
      </el-table-column>
      <el-table-column prop="idCard" label="身份证号" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column prop="email" label="邮箱" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column prop="status" label="状态" sortable="custom" header-align="center" align="center">
        <template v-slot="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">停用</el-tag>
          <el-tag v-else size="small" type="success">正常</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('sys:user:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="state.hasPermission('sys:user:delete')" type="danger" link @click="state.deleteHandle(scope.row.id)">删除</el-button>
          <el-button type="warning" link @click="resetPassword(scope.row.id)">重置密码</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { reactive, ref, toRefs ,inject,onMounted} from "vue";
import AddOrUpdate from "./doctorUser-add-or-update.vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";

const httpUrl = inject('httpUrl'); // 注入全局变量
const emit = defineEmits(["refreshDataList"]);

const officeList = ref<any[]>([]);

const view = reactive({
  getDataListURL: "/sys/user/doctorPage",
  getDataListIsPage: true,
  deleteURL: "/sys/user",
  deleteIsBatch: true,
  exportURL: "/sys/user/exportMrmsUser",
  dataForm: {
    username: "",
    officeId: null, 
    gender: "" ,
    userType:1
  }
});

onMounted(()=>{
  getOfficeList();
}) 
// 获取科室列表
const getOfficeList = () => {
  return baseService.get("/sys/office/list").then((res) => {
    officeList.value = res.data;
  });
};
const state = reactive({ ...useView(view), ...toRefs(view) });


const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
};
const resetPassword = (id:number) => {
  baseService.put("/sys/user/resetPassword/"+id).then((res) => {
    ElMessage.success({
      message: "重置密码成功",
      duration: 500,
      onClose: () => {
        emit("refreshDataList");
      }
    })
  }
)} 
</script>
<style>
.user-avatar{
  width: 80px;
  height: 80px;
}
</style>