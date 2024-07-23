<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="设备号" prop="sn">
        <el-input
          v-model="queryParams.sn"
          placeholder="请输入设备号"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="预约价格" prop="price">
        <el-input
          v-model="queryParams.price"
          placeholder="请输入预约价格"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="在线状态" prop="onlineStatus">
        <el-select v-model="queryParams.onlineStatus" placeholder="请选择在线状态" clearable style="width: 150px;">
          <el-option
            v-for="dict in robot_online_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="启用状态" prop="enableStatus">
        <el-select v-model="queryParams.enableStatus" placeholder="请选择启用状态" clearable style="width: 150px;">
          <el-option
            v-for="dict in robot_enable_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="创建者" prop="createBy">
        <el-input
          v-model="queryParams.createBy"
          placeholder="请输入创建者"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="更新者" prop="updateBy">
        <el-input
          v-model="queryParams.updateBy"
          placeholder="请输入更新者"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['qd:robot:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['qd:robot:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['qd:robot:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['qd:robot:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="robotList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="设备号" align="center" prop="sn" />
      <el-table-column label="描述" align="center" prop="title" />
      <el-table-column label="机器人照片" align="center" prop="image" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.image" :width="50" :height="50"/>
        </template>
      </el-table-column>
      <el-table-column label="预约价格" align="center" prop="price" />
      <el-table-column label="在线状态" align="center" prop="onlineStatus">
        <template #default="scope">
          <dict-tag :options="robot_online_status" :value="scope.row.onlineStatus"/>
        </template>
      </el-table-column>
      <el-table-column label="启用状态" align="center" prop="enableStatus">
        <template #default="scope">
          <dict-tag :options="robot_enable_status" :value="scope.row.enableStatus"/>
        </template>
      </el-table-column>
      <el-table-column label="创建者" align="center" prop="createBy" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新者" align="center" prop="updateBy" />
      <el-table-column label="更新时间" align="center" prop="updateTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.updateTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['qd:robot:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['qd:robot:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改机器人对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="robotRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="设备号" prop="sn">
          <el-input v-model="form.sn" placeholder="请输入设备号" />
        </el-form-item>
        <el-form-item label="描述" prop="title">
          <el-input v-model="form.title" placeholder="请输入描述" />
        </el-form-item>
        <el-form-item label="机器人照片" prop="image">
          <image-upload v-model="form.image"/>
        </el-form-item>
        <el-form-item label="预约价格" prop="price">
          <el-input v-model="form.price" placeholder="请输入预约价格" />
        </el-form-item>
        <el-form-item label="在线状态" prop="onlineStatus">
          <el-radio-group v-model="form.onlineStatus">
            <el-radio
              v-for="dict in robot_online_status"
              :key="dict.value"
              :label="dict.value"
            >{{dict.label}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="启用状态" prop="enableStatus">
          <el-radio-group v-model="form.enableStatus">
            <el-radio
              v-for="dict in robot_enable_status"
              :key="dict.value"
              :label="dict.value"
            >{{dict.label}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" type="textarea" placeholder="请输入内容" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Robot">
import {listRobot, getRobot, delRobot, addRobot, updateRobot} from "@/api/qd/robot";

const {proxy} = getCurrentInstance();
const {robot_online_status, robot_enable_status} = proxy.useDict('robot_online_status', 'robot_enable_status');

const robotList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    sn: null,
    image: null,
    price: null,
    onlineStatus: null,
    enableStatus: null,
    createBy: null,
    updateBy: null,
  },
  rules: {
    sn: [
      {required: true, message: "设备号不能为空", trigger: "blur"}
    ],
    title: [
      {required: true, message: "描述不能为空", trigger: "blur"}
    ],
    price: [
      {required: true, message: "预约价格不能为空", trigger: "blur"}
    ],
    onlineStatus: [
      {required: true, message: "在线状态不能为空", trigger: "change"}
    ],
    enableStatus: [
      {required: true, message: "启用状态不能为空", trigger: "change"}
    ],
  }
});

const {queryParams, form, rules} = toRefs(data);

/** 查询机器人列表 */
function getList() {
  loading.value = true;
  listRobot(queryParams.value).then(response => {
    robotList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    sn: null,
    title: null,
    image: null,
    price: null,
    onlineStatus: null,
    enableStatus: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
    remark: null
  };
  proxy.resetForm("robotRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加机器人";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getRobot(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改机器人";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["robotRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateRobot(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addRobot(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal.confirm('是否确认删除机器人编号为"' + _ids + '"的数据项？').then(function () {
    return delRobot(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {
  });
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('qd/robot/export', {
    ...queryParams.value
  }, `robot_${new Date().getTime()}.xlsx`)
}

getList();
</script>
