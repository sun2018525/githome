
<template>
  <div class="announcement-container">
    <!-- 头部区域 -->
    <div class="header-section">
      <div>
        <h1 class="page-title">欢迎登录电子病历管理系统</h1>
      </div>
    </div>

    <!-- 筛选区域 -->
    <div class="filter-section">
     <el-form :inline="true" ref="dataFormRef" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-row :gutter="18" align="middle">
        <el-col :span="10">
              <el-input v-model="state.dataForm.title" placeholder="搜索公告标题" clearable>
              </el-input> 
            <template #prefix>
            <el-icon><Search /></el-icon>
          </template>
        </el-col> 
        <el-col :span="8">
          <el-button type="primary" icon="Search" @click="state.getDataList()">搜索</el-button>
          <el-button icon="Refresh" @click="reset()">重置</el-button>
        </el-col>
      </el-row>
        </el-form>
    </div>

    <!-- 公告列表 -->
    <div class="announcement-list">
      <el-card 
        v-for="notice in state.dataList" 
        :key="notice.id"
        class="announcement-card" 
      >
        <div class="announcement-item">
          <div class="announcement-icon">
             <el-icon v-if="notice.noticeType === '1'" class="priority-high" size="20">
              <Bell />
            </el-icon>
            <el-icon v-else-if="notice.noticeType === '0'" class="priority-medium" size="20">
              <InfoFilled />
            </el-icon>
            <el-icon v-else  class="priority-low" size="20">
              <Document />
            </el-icon>
          </div>
          
          <div class="announcement-content">
            <div class="announcement-title">{{ notice.title }}</div>
            <div class="announcement-desc">{{ notice.content }}</div>
            
            <div class="announcement-meta">
              <span class="meta-item">
                <el-icon><Clock /></el-icon>
                {{ notice.createDate }}
              </span> 
              <el-tag 
                class="status-tag"
                >已发布</el-tag> 
            </div>
          </div>
           
        </div>
      </el-card>
    </div>

    <!-- 分页 -->
    <div class="pagination-section">
      <el-pagination
        :current-page="state.page" 
        background
        layout="prev, pager, next, total"
        :page-size="state.limit" 
        :total="state.total"
        @size-change="state.pageSizeChangeHandle" 
        @current-change="state.pageCurrentChangeHandle"
      />
    </div>
  </div>
</template>
<script setup lang="ts">
  
import useView from "@/hooks/useView";
import { reactive, toRefs,ref} from "vue";
import {
  ElCard,
  ElButton,
  ElInput, 
  ElPagination,
  ElTag,
  ElIcon,
  ElRow,
  ElCol
} from 'element-plus'
import { 
  Search, 
  Bell, 
  Clock 
} from '@element-plus/icons-vue'

const view = reactive({
  getDataListURL: "/sys/sysnotice/page",
  getDataListIsPage: true,
  dataForm:{
    title:"",
    isPublish:1,
    noticeType:""
  }
});
const state = reactive({ ...useView(view), ...toRefs(view) });
const reset=()=>{
  state.dataForm.title="";
}
</script>

<style scoped>
.announcement-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.page-title {
  font-size: 24px;
  color: #303133;
  margin: 0;
}

.page-subtitle {
  color: #909399;
  margin: 4px 0 0 0;
}

.filter-section {
  background: #f5f7fa;
  padding: 16px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.announcement-card {
  transition: all 0.3s ease;
  margin-bottom: 16px;
  border-radius: 8px;
  overflow: hidden;
}

.announcement-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.priority-high {
  border-left: 4px solid #f56c6c;
}

.priority-medium {
  border-left: 4px solid #e6a23c;
}

.priority-low {
  border-left: 4px solid #909399;
}

.announcement-item {
  display: flex;
  align-items: flex-start;
  gap: 16px;
}

.announcement-icon {
  margin-top: 2px;
}

.announcement-content {
  flex: 1;
}

.announcement-title {
  font-size: 16px;
  font-weight: 600;
  color: #303133;
  margin-bottom: 8px;
}

.announcement-desc {
  color: #606266;
  line-height: 1.5;
  margin-bottom: 12px;
}

.announcement-meta {
  display: flex;
  align-items: center;
  gap: 16px;
  color: #909399;
  font-size: 12px;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 4px;
}

.status-tag {
  margin-left: auto;
}

.announcement-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.pagination-section {
  display: flex;
  justify-content: center;
  margin-top: 24px;
}
</style>
