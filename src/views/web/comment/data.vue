<template>
  <div class="app-container">
    <el-table
      v-loading="loading"
      :data="commentList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="序号" align="center" prop="id" />
      <el-table-column label="头像" align="center" prop="avatar">
        <template #default="scope">
          <el-avatar :size="60" :src="scope.row.avatar"> </el-avatar>
        </template>
      </el-table-column>
      <el-table-column label="评论人" align="center" prop="uid" />
      <el-table-column label="被评论人" align="center" prop="noteUid" />
      <el-table-column label="评论内容" align="center" prop="content" />
      <el-table-column label="评论等级" align="center" prop="level" />
      <el-table-column
        label="评论时间"
        align="center"
        prop="createTime"
        width="180"
      >
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        fixed="right"
        min-width="200"
      >
        <template #default="scope">
          <el-tooltip content="删除" placement="top">
            <el-button
              type="danger"
              icon="Delete"
              size="small"
              @click="handleDelete(scope.row)"
              v-hasPermi="['web:member:remove']"
            />
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />
  </div>
</template>

<script setup name="Comment">
import { getCommentByNid, delComment } from "@/api/web/comment";

const { proxy } = getCurrentInstance();

const commentList = ref([]);
const loading = ref(true);
const ids = ref([]);
const total = ref(0);

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    uid: undefined,
    noteUid: undefined,
    content: undefined,
  },
  rules: {},
});

const { queryParams, form, rules } = toRefs(data);

/** 查询评论列表 */
function getList() {
  loading.value = true;
  getCommentByNid(queryParams.value).then((response) => {
    commentList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const commentIds = row.id || ids.value;
  proxy.$modal
    .confirm('是否确认删除评论编号为"' + commentIds + '"的数据项？')
    .then(function () {
      return delComment(commentIds);
    })
    .then(() => {
      getList();
      proxy.$modal.msgSuccess("删除成功");
    })
    .catch(() => {});
}
getList();
</script>
