<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-12-23 09:34
 * @LastAuthor : itchaox
 * @LastTime   : 2024-01-09 01:00
 * @desc       : 
-->

<script setup lang="ts">
  import { bitable } from '@lark-base-open/js-sdk';
  import Drawer from './Drawer.vue';
  import { LOCAL_STORAGE_KEY } from '@/config/constant';

  // 新增视图抽屉
  const addViewDrawer = ref(false);

  async function addView() {
    drawerStatus.value = 'add';
    addViewDrawer.value = true;
  }

  onMounted(() => {
    // 检索存储的数据
    const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

    if (storedData) {
      // 已存在 key
      const retrievedArray = JSON.parse(storedData);
      methodList.value = retrievedArray;
      filterTableDataList.value = retrievedArray;
    }
  });

  const methodList = ref([]);

  const confirmAddView = (item) => {
    // debugger;
    if (drawerStatus.value === 'add') {
      methodList.value.push(item);
    } else {
      // 新增则 push 数据, 编辑修改数据
      methodList.value = methodList.value.map((_item) => {
        if (_item.id === item.id) {
          _item = item;
        }
        return _item;
      });
      filterTableDataList.value = methodList.value;
    }

    // console.log('methodList.value', methodList.value);
  };

  function cancelAddView() {
    addMethodItem.value = '';
  }

  const handleDelete = (index, id) => {
    methodList.value.splice(index, 1);

    // 检索存储的数据
    const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

    if (storedData) {
      // 已存在 key
      const retrievedArray = JSON.parse(storedData);
      const _filter = retrievedArray.filter((item) => item.id !== id);
      localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(_filter));

      methodList.value = _filter;
    }

    ElMessage({
      type: 'success',
      message: '删除成功',
      duration: 1500,
      showClose: true,
    });
  };

  const selectList = ref([]);

  const handleSelectionChange = (val) => {
    selectList.value = val;
  };

  let table;
  let view;

  async function init() {
    table = await bitable.base.getActiveTable();
    view = await table.getActiveView();
  }

  onMounted(() => {
    init();
  });

  bitable.base.onSelectionChange(async (event) => {
    init();
  });

  async function use(item) {
    const _sortList = await view.getSortInfo();
    if (_sortList?.length > 0) {
      for (const item of _sortList) {
        await view.deleteSort(item);
      }
      await view.applySetting();
    }

    // 筛选
    if (item?.list?.length > 0) {
      await view.addSort(
        item?.list?.map((item) => ({
          fieldId: item.id,
          desc: item.desc,
        })),
      );
      await view.applySetting();
    }
  }

  const addMethodItem = ref();

  async function edit(item) {
    addMethodItem.value = item;
    drawerStatus.value = 'edit';
    addViewDrawer.value = true;
  }

  function batchDelete() {
    if (selectList.value.length === 0) {
      ElMessage({
        type: 'warning',
        message: '请选择删除的方案!',
        duration: 1500,
        showClose: true,
      });
      return;
    }

    const difference = methodList.value.filter((method) => !selectList.value.some((select) => select.id === method.id));

    methodList.value = difference;
    filterTableDataList.value = difference;

    // 更新存储的数据
    localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(difference));

    ElMessage({
      type: 'success',
      message: '批量删除成功',
      duration: 1500,
      showClose: true,
    });
  }

  const loading = ref(false);

  const drawerStatus = ref('add');

  const methodName = ref('');

  const filterTableDataList = ref();

  function search() {
    // 先重新获取全部数据
    filterTableDataList.value = methodList.value;

    // 筛选插件信息
    filterTableDataList.value = filterTableDataList.value.filter((item) => {
      let _name = item.name;
      const nameMatch = !methodName.value || _name?.includes(methodName.value);
      return nameMatch;
    });

    ElMessage({
      type: 'success',
      message: '查询成功',
      duration: 1500,
      showClose: true,
    });
  }

  function reset() {
    methodName.value = '';
    filterTableDataList.value = methodList.value;
  }
</script>

<template>
  <div class="tip">
    <div class="tip-text tip-title">操作步骤:</div>
    <div class="tip-text">1. 新增方案: 配置方案名字和排序条件</div>
    <div class="tip-text">2. 点击"运行"按钮，使用当前方案的排序条件</div>
    <div class="tip-text">3. 点击"编辑"按钮，修改选定方案名字和排序条件</div>
  </div>

  <div
    v-loading="loading"
    element-loading-text="加载中..."
  >
    <div class="button">
      <el-button
        type="primary"
        @click="addView"
        size="small"
      >
        <el-icon><Plus /></el-icon>
        <span>新增方案</span>
      </el-button>
    </div>

    <div class="addView-line">
      <div class="addView-line-label">方案名字:</div>
      <el-input
        style="width: 60%"
        v-model="methodName"
        clearable
        placeholder="请输入方案名字"
        @keydown.enter="search"
      />
    </div>

    <div class="button">
      <el-button
        type="primary"
        size="small"
        @click="search"
      >
        <el-icon><Search /></el-icon>
        <span>查询</span>
      </el-button>

      <el-button
        type="info"
        size="small"
        @click="reset"
      >
        <el-icon><Refresh /></el-icon>
        <span>重置</span>
      </el-button>
    </div>

    <div class="view-table">
      <el-table
        ref="tableRef"
        @selection-change="handleSelectionChange"
        :data="filterTableDataList"
        height="100%"
        empty-text="暂无数据"
      >
        <el-table-column
          type="selection"
          width="30"
        />

        <el-table-column
          label="方案名字"
          :min-width="120"
        >
          <template #default="scope">
            <div
              :title="scope.row.name"
              class="view-name"
            >
              <div>{{ scope.row.name }}</div>
            </div>
          </template>
        </el-table-column>
        <el-table-column
          property="name"
          label="操作"
          align="center"
          width="100"
        >
          <template #default="scope">
            <div class="operation">
              <div
                class="cursor-pointer"
                @click="use(scope.row)"
                title="运用排序条件"
                style="color: rgb(20, 86, 240)"
              >
                <el-icon size="20"><VideoPlay /></el-icon>
              </div>
              <div
                class="cursor-pointer"
                @click="edit(scope.row)"
                title="编辑方案"
              >
                <el-icon size="20"><Edit /></el-icon>
              </div>

              <div
                class="cursor-pointer"
                @click="handleDelete(scope.$index, scope.row.id)"
                title="删除方案"
                style="color: #f54a45"
              >
                <el-icon size="20"><Delete /></el-icon>
              </div>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="delete-button">
      <el-button
        @click="batchDelete"
        type="danger"
        size="small"
        color="#F54A45"
      >
        <el-icon><Delete /></el-icon>
        <span>批量删除</span>
      </el-button>
    </div>
  </div>

  <Drawer
    v-model:model-value="addViewDrawer"
    :method-list="methodList"
    :addMethodItem="addMethodItem"
    :drawerStatus="drawerStatus"
    @confirmAddView="confirmAddView"
    @cancel-add-view="cancelAddView"
  />
</template>

<style scoped>
  .home {
    font-size: 14px;
  }

  .delete-button {
    margin-top: 20px;
  }

  .addView {
    margin: 10px 0;
  }

  .addView-line {
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    .addView-line-label {
      margin-right: 10px;
      font-size: 14px;
      white-space: nowrap;
    }
  }

  .operation {
    display: flex;

    div {
      margin-right: 12px;
    }
  }

  .tip {
    color: #8f959e;
    font-size: 12px;
    margin-bottom: 14px;
    .tip-text {
      margin-bottom: 6px;
    }

    .tip-info {
      margin-top: 12px;
    }

    .tip-title {
      font-size: 14px;
      margin-bottom: 8px;
    }
  }

  .button {
    margin: 14px 0;
  }

  .cursor-pointer {
    cursor: pointer;
  }
</style>
