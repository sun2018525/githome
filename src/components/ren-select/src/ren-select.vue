
<template>
  <el-select 
    v-model="value" 
    @change="handleChange" 
    :placeholder="placeholder" 
    clearable 
    @clear="handleClear"
  >
    <el-option 
      :label="data.dictLabel" 
      v-for="data in dataList" 
      :key="data.dictValue" 
      :value="data.dictValue"
    />
  </el-select>
</template>

<script lang="ts">
import { computed, defineComponent } from "vue";
import { getDictDataList } from "@/utils/utils";
import { useAppStore } from "@/store";

export default defineComponent({
  name: "RenSelect",
  emits: ['update:modelValue', 'clear'],
  props: {
    modelValue: [Number, String],
    dictType: String,
    placeholder: String
  },
  setup(props, { emit }) {
    const store = useAppStore();
    const value = computed({
      get: () => props.modelValue,
      set: (val) => emit('update:modelValue', val)
    });
    
    const dataList = getDictDataList(store.state.dicts, props.dictType);
    
    const handleChange = (value: any) => {
      emit('update:modelValue', value);
    };
    
    const handleClear = () => {
      // 清空后的自定义逻辑
      console.log('选择器已被清空');
      emit('clear'); // 正确触发clear事件
    };
    
    return {
      value,
      dataList,
      handleChange,
      handleClear
    };
  }
});
</script>
