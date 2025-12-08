<template>
  <div class="mod-demo__sysoffice">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.officeName" placeholder="科室名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:office:page')" @click="state.getDataList()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="danger" v-if="state.hasPermission('sys:office:delete')" @click="state.deleteHandle()">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
              <el-table-column prop="id" label="主键" header-align="center" align="center"></el-table-column>
              <el-table-column prop="officeName" label="科室名称" header-align="center" align="center"></el-table-column>
              <el-table-column prop="officePic" label="科室图片" header-align="center" align="center">
                 <template #default="{ row }">
                  <img
                    v-if="row.officePic"
                    :src="httpUrl+row.officePic"
                    class="user-avatar"
                  />
                <span v-else class="no-avatar">无头像</span>
                </template>
              </el-table-column>
              <el-table-column prop="officeAddr" label="科室位置" header-align="center" align="center"></el-table-column>
              <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
              <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column>
            <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button type="primary" link @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="state.hasPermission('sys:office:delete')" type="danger" link @click="state.deleteHandle(scope.row.id)">删除</el-button>
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

import AddOrUpdate from "./office-add-or-update.vue";
const httpUrl = inject('httpUrl'); // 注入全局变量
const view = reactive({
  getDataListURL: "/sys/office/page",
  getDataListIsPage: true,
  deleteURL: "/sys/office",
  deleteIsBatch: true,
    dataForm: {
      officeName: "", 
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
};

</script>
<style>
.user-avatar{
  width: 80px;
  height: 80px;
}
</style>