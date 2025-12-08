<template>
  <div class="mod-demo__sysnotice">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item label="是否发布" prop="isPublish">
        <el-select v-model="state.dataForm.isPublish" placeholder="是否发布" clearable>
          <el-option label="否" :value="0"></el-option>
          <el-option label="是" :value="1"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="公告类型" prop="noticeType">
        <ren-select v-model="state.dataForm.noticeType" dict-type="notice_type" placeholder="公告类型" ></ren-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="danger" @click="state.deleteHandle()">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
              <el-table-column prop="title" label="公告标题" header-align="center" align="center"></el-table-column>
              <el-table-column prop="content" label="公告内容" header-align="center" align="center"></el-table-column>
              <el-table-column prop="noticeType" label="公告类型" header-align="center" align="center">
                <template v-slot="scope">
                  {{ state.getDictLabel("notice_type", scope.row.noticeType) }}
                </template>
              </el-table-column>
              <el-table-column prop="isPublish" label="是否发布" header-align="center" align="center">
                <template v-slot="scope">
                  <el-tag v-if="scope.row.isPublish === 1" size="small">是</el-tag>
                  <el-tag v-else size="small" type="danger">否</el-tag>
                </template>
              </el-table-column>
              <!-- <el-table-column prop="creator" label="创建者" header-align="center" align="center"></el-table-column> -->
              <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
              <!-- <el-table-column prop="updater" label="更新者" header-align="center" align="center"></el-table-column> -->
              <!-- <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column> -->
            <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button type="success" :disabled="scope.row.isPublish===1" link @click="publish(scope.row.id)">发布</el-button>
          <el-button type="primary" :disabled="scope.row.isPublish===1" link @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
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
import { reactive, ref, toRefs } from "vue";
import baseService from "@/service/baseService";
import { ElMessage, ElMessageBox } from "element-plus";
import AddOrUpdate from "./sysnotice-add-or-update.vue";

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/sys/sysnotice/page",
  getDataListIsPage: true,
  exportURL: "/sys/sysnotice/export",
  deleteURL: "/sys/sysnotice",  
  dataForm: {
    title:null,
    isPublish: null,
    noticeType:null
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
};
// 表单提交
const publish = (id:number) => {
  ElMessageBox.confirm("确定进行[发布]操作?", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
    })
    .then(() => {
      state.dataListLoading = true;
      baseService.put("/sys/sysnotice",{id:id,isPublish:1}).then((res) => {
      ElMessage.success({
        message: '发布成功',
        duration: 500,
        onClose: () => {
          state.dataListLoading = false;
          state.getDataList();
        }
      });
    });
  });
};
</script>
