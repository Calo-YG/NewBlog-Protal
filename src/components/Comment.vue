<template>
  <el-dialog
    title="评论"
    width="60%"
    v-model="dialogDodel"
    @close="cancel"
  >
    <el-form>
      <el-form-item>
        <el-input
          type="textarea"
          v-model="content"
          placeholder="文明社会，理性评论"
          autocomplete="off"
        ></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="default" @click="cancel">取消</el-button>
      <el-button type="primary" @click="handleOk">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script lang="ts" setup>
import { ElMessage } from "element-plus";
import { ref, watch } from "vue";

/**
 * 定义props
 */
const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  comment_id: {
    type: String,
    default: "",
  },
  article_id: {
    type: String,
    default: "",
  },
  to_user: {
    // type: any,
    default: {},
  },
});

//#region 定义响应式数据
const dialogDodel = ref(false);
const btnLoading = ref(false);
const content = ref("");
const cacheTime = ref(0);
const times = ref(0);
//#endregion
const cancel = (): boolean => {
  return false;
};

const handleOk = async (): Promise<void> => {
  ElMessage({
    message: "操作成功",
    type: "success",
  });
};

watch(props, (val, oldVal) => {
  dialogDodel.value = val.visible;
});
</script>
<style scoped>
.dialog-footer {
  text-align: right;
}
</style>
