<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
          <el-form-item label="公告标题" prop="title">
        <el-input v-model="dataForm.title" placeholder="公告标题"></el-input>
      </el-form-item>
        <el-form-item label="公告内容" prop="content">
        <el-input type="textarea" v-model="dataForm.content" placeholder="公告内容"></el-input>
      </el-form-item>
      <el-form-item label="公告类型" prop="noticeType">
        <ren-select v-model="dataForm.noticeType" dict-type="notice_type"></ren-select>
      </el-form-item>
      <el-form-item label="是否发布" prop="isPublish">
        <el-select v-model="dataForm.isPublish" placeholder="是否发布" clearable>
          <el-option label="否" :value="0"></el-option>
          <el-option label="是" :value="1"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: '',  title: '',  content: '',  noticeType: '',  isPublish: '',  creator: '',  createDate: '',  updater: '',  updateDate: ''});

const rules = ref({
          title: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          content: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          noticeType: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          isPublish: [
      { required: true, message: '必填项不能为空', trigger: 'blur' }
    ],
          });

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  if (id) {
    getInfo(id);
  }
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/sys/sysnotice/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/sysnotice", dataForm).then((res) => {
      ElMessage.success({
        message: '保存成功',
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};

defineExpose({
  init
});
</script>
