<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryRef"
      :inline="true"
      v-show="showSearch"
    >
      <el-form-item label="评论人" prop="uid">
        <el-input
          v-model="queryParams.uid"
          placeholder="请输入评论人"
          clearable
          style="width: 200px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="被评论人" prop="noteUid">
        <el-input
          v-model="queryParams.noteUid"
          placeholder="请输入被评论人"
          clearable
          style="width: 200px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="被评论笔记" prop="nid">
        <el-input
          v-model="queryParams.nid"
          placeholder="请输入被评论笔记"
          clearable
          style="width: 200px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="评论内容" prop="content">
        <el-input
          v-model="queryParams.content"
          placeholder="请输入评论内容"
          clearable
          style="width: 200px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="评论时间" style="width: 308px">
        <el-date-picker
          v-model="dateRange"
          value-format="YYYY-MM-DD"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery"
          >搜索</el-button
        >
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

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
      <el-table-column label="评论人" align="center" prop="username" />
      <el-table-column label="被评论人" align="center" prop="pushUsername" />
      <el-table-column label="被评论笔记" align="center" prop="title">
        <template #default="scope">
          <el-link
            type="primary"
            :underline="false"
            @click="handleDetail(scope.row)"
          >
            <el-tooltip
              class="item"
              effect="dark"
              :content="scope.row.title"
              placement="top"
            >
              <div class="table-cell">{{ scope.row.title }}</div>
            </el-tooltip>
          </el-link>
        </template>
      </el-table-column>

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
import { listComment, delComment } from "@/api/web/comment";
import { useRouter } from "vue-router";

const { proxy } = getCurrentInstance();
const router = useRouter();

const commentList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const dateRange = ref([]);

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

const { queryParams, form } = toRefs(data);

/** 查询评论列表 */
function getList() {
  loading.value = true;
  listComment(proxy.addDateRange(queryParams.value, dateRange.value)).then(
    (response) => {
      commentList.value = response.rows;
      total.value = response.total;
      loading.value = false;
    }
  );
}
/** 查看详情操作 */
function handleDetail(row) {
  router.push({ path: "/message/data" });
}
/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}
/** 重置按钮操作 */
function resetQuery() {
  dateRange.value = [];
  proxy.resetForm("queryRef");
  handleQuery();
}
/** 多选框选中数据 */
function handleSelectionChange(selection) {
  ids.value = selection.map((item) => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
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
