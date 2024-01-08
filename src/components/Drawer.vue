<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-12-16 09:57
 * @LastAuthor : itchaox
 * @LastTime   : 2024-01-09 00:51
 * @desc       : 抽屉
-->

<script setup lang="ts">
  import { bitable } from '@lark-base-open/js-sdk';
  import { Close } from '@element-plus/icons-vue';
  import FieldIcon from './fieldIcon.jsx';
  import { v4 as uuidv4 } from 'uuid';
  import { LOCAL_STORAGE_KEY } from '@/config/constant';

  import { Drag } from '@icon-park/vue-next';

  import { VueDraggable } from 'vue-draggable-plus';

  const base = bitable.base;

  interface Props {
    modelValue: Boolean;
    methodList: any[];
    addMethodItem?: any;
    drawerStatus?: any;
  }

  const props = defineProps<Props>();

  const emits = defineEmits(['update:model-value', 'confirmAddView', 'cancelAddView']);

  let table;
  let view;

  // 字段列表
  const fieldList = ref([]);

  const addMethodName = ref();

  // FIXME 排序
  const sortList = ref([]);

  /**
   * @desc  : 确认新增/修改方案
   */
  async function confirmAddView() {
    console.log('sortList.value', sortList.value);

    if (!addMethodName.value) {
      ElMessage({
        type: 'error',
        message: '请填写方案名字',
        duration: 1500,
        showClose: true,
      });
      return;
    }

    const index = props.methodList
      .filter((i) => i.name !== props.addMethodItem?.name)
      .findIndex((item) => item.name === addMethodName.value);
    if (index === -1) {
      // addMethodName.value = '';
      // addViewType.value = 1;
      // emits('update:model-value', false);

      let item = {
        // 处理 id
        id: props.drawerStatus === 'add' ? uuidv4() : props.addMethodItem?.id,
        name: addMethodName.value,
        list: sortList.value,
      };

      // 检索存储的数据
      const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

      if (storedData) {
        // 已存在 key
        let retrievedArray = JSON.parse(storedData);

        if (props.drawerStatus === 'add') {
          retrievedArray.push(item);
        } else {
          // 新增则 push 数据, 编辑修改数据
          retrievedArray = retrievedArray.map((_item) => {
            if (_item.id === item.id) {
              _item = item;
            }
            return _item;
          });
        }

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(retrievedArray));
      } else {
        // 不存在存在 key

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify([item]));
      }

      reset();
      emits('confirmAddView', item);

      ElMessage({
        type: 'success',
        message: `${props.drawerStatus === 'add' ? '新增' : '编辑'}成功`,
        duration: 1500,
        showClose: true,
      });
    } else {
      ElMessage({
        type: 'error',
        message: '方案名字已存在,请重新输入!',
        duration: 1500,
        showClose: true,
      });
    }
  }

  function cancel() {
    reset();
  }

  function reset() {
    emits('update:model-value', false);
    emits('cancelAddView');

    addMethodName.value = '';
    sortList.value = [];
  }

  watch(
    () => props.addMethodItem,
    (newValue, oldValue) => {
      // debugger;
      if (newValue) {
        console.log('🚀  newValue:', newValue);

        addMethodName.value = newValue?.name;
        sortList.value = newValue?.list || [];
      }
    },
    {
      immediate: true,
      deep: true,
    },
  );

  // 折叠面板
  const collapse = ref('1');

  const addSort = () => {
    const idsInGroup = sortList.value.map((item) => item.id);
    const _list = sortFieldList.value.filter((item) => !idsInGroup.includes(item.id));

    if (_list?.[0]) {
      sortList.value.push({
        id: _list?.[0]?.id,
        name: _list?.[0]?.name,
        type: _list?.[0]?.type,
        desc: false,
      });
    }
  };

  const sortFiledChange = async (item, index) => {
    let _activeItem = sortFieldList.value.find((i) => i.id === item.id);

    sortList.value[index] = {
      name: _activeItem.name,
      type: _activeItem.type,
      id: _activeItem.id,
      desc: false,
    };
  };

  // 排序的字段列表
  const sortFieldList = ref([]);

  async function init() {
    table = await base.getActiveTable();
    view = await table.getActiveView();
    fieldList.value = await view.getFieldMetaList();

    sortFieldList.value = fieldList.value;
  }

  onMounted(async () => {
    init();
  });

  base.onSelectionChange(async (event) => {
    init();
  });

  /**
   * @desc  : 顺序
   * @param  {any} type：字段类型
   * @return {any} 文本
   */
  const getGroupTextOrder = (type) => {
    let textList = [1, 11, 13, 15, 17, 18, 19, 20, 21, 22, 23, 403, 1003, 1004, 99001];
    let numberList = [2, 5, 1001, 1002, 1005, 99002, 99003, 99004];
    let optionList = [3, 4, 7];

    let _text;
    if (textList.includes(type)) {
      _text = 'A → Z';
    } else if (numberList.includes(type)) {
      _text = '0 → 9';
    } else if (optionList.includes(type)) {
      _text = '选项顺序';
    } else {
      _text = 'A → Z';
    }

    return _text;
  };

  /**
   * @desc  : 倒序
   * @param  {any} type：字段类型
   * @return {any} 文本
   */
  const getGroupTextReverseOrder = (type) => {
    let textList = [1, 11, 13, 15, 17, 18, 19, 20, 21, 22, 23, 403, 1003, 1004, 99001];
    let numberList = [2, 5, 1001, 1002, 1005, 99002, 99003, 99004];
    let optionList = [3, 4, 7];

    let _text;
    if (textList.includes(type)) {
      _text = 'Z → A';
    } else if (numberList.includes(type)) {
      _text = '9 → 0';
    } else if (optionList.includes(type)) {
      _text = '选项倒序';
    } else {
      _text = 'Z → A';
    }
    return _text;
  };

  const el2 = ref();
</script>

<template>
  <div>
    <el-drawer
      :model-value="modelValue"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      @closed="cancel"
      :show-close="false"
      size="85%"
    >
      <template #header="{ close, titleId }">
        <div
          :id="titleId"
          v-if="drawerStatus === 'add'"
        >
          新增方案
        </div>
        <div
          :id="titleId"
          v-else
        >
          编辑方案
        </div>
        <el-button
          type="danger"
          @click="close"
          size="small"
        >
          <el-icon class="el-icon--left"><CircleCloseFilled /></el-icon>
          关闭
        </el-button>
      </template>

      <div class="addView">
        <div class="addView-line">
          <div class="addView-line-label theme-view-text-color">方案名字:</div>
          <el-input
            style="width: 60%"
            clearable
            v-model="addMethodName"
            placeholder="请输入方案名字"
          />
        </div>

        <el-collapse
          v-model="collapse"
          class="collapse"
        >
          <!-- FIXME 排序 -->
          <el-collapse-item name="1">
            <template #title>
              <AlphabeticalSorting theme="outline" />
              <span class="collapse-title">设置排序条件</span>
              <span
                v-if="sortList.length > 0"
                style="color: #4493c5"
                >（{{ sortList.length }}）</span
              >
            </template>
            <VueDraggable
              ref="el2"
              v-model="sortList"
              animation="150"
              handle=".handle2"
              class="collapse-line-list"
            >
              <!-- <div class="collapse-line-list"> -->

              <div
                class="collapse-line"
                v-for="(item, index) in sortList"
                :key="item.id"
              >
                <div class="drag2">
                  <Drag
                    class="handle2 cursor-move"
                    theme="outline"
                    size="18"
                    fill="#333"
                    strokeLinejoin="miter"
                    strokeLinecap="butt"
                  />
                  <!-- 字段名 -->
                  <div class="collapse-line-filed">
                    <el-select
                      size="small"
                      v-model="item.id"
                      :title="item.name"
                      @change="sortFiledChange(item, index)"
                    >
                      <el-option
                        v-for="field in sortFieldList"
                        :key="field.id"
                        :label="field.name"
                        :title="field.name"
                        :value="field.id"
                        :disabled="sortList.map((j) => j.id).includes(field.id)"
                      >
                        <field-icon :fieldType="field.type" />
                        <span>
                          {{ field.name }}
                        </span>
                      </el-option>
                    </el-select>
                  </div>
                </div>

                <div class="collapse-line-other">
                  <!-- 值 -->
                  <div class="collapse-line-value">
                    <el-button-group size="small">
                      <el-button
                        class="collapse-btn"
                        type="primary"
                        :plain="item.desc"
                        @click="() => (item.desc = !item.desc)"
                        >{{ getGroupTextOrder(item.type) }}</el-button
                      >
                      <el-button
                        class="collapse-btn"
                        type="primary"
                        :plain="!item.desc"
                        @click="() => (item.desc = !item.desc)"
                      >
                        {{ getGroupTextReverseOrder(item.type) }}
                      </el-button>
                    </el-button-group>
                  </div>
                  <el-button
                    :icon="Close"
                    class="collapse-delete"
                    @click="() => sortList.splice(index, 1)"
                    text
                  />
                </div>
              </div>
              <!-- </div> -->
            </VueDraggable>
            <el-button
              v-if="sortFieldList.length > sortList.length"
              text
              @click="addSort"
            >
              <el-icon><Plus /></el-icon>添加条件
            </el-button>
          </el-collapse-item>
        </el-collapse>

        <div>
          <el-button
            type="primary"
            @click="confirmAddView"
            >确定</el-button
          >

          <el-button
            type="info"
            @click="cancel"
            >取消</el-button
          >
        </div>
      </div>
    </el-drawer>
  </div>
</template>

<style scoped>
  .addView {
    /* margin: 10px 0; */
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

    .addView-line-labelDialog {
      width: 125px;
    }
  }

  .collapse {
    margin: 20px 0;
  }

  .collapse-title {
    margin-left: 5px;
    position: relative;
    bottom: 1px;
  }

  .collapse-line-list {
    margin: 10px 0;
  }

  .collapse-line {
    display: flex;
    flex-wrap: nowrap;
    justify-content: space-between;
    margin-bottom: 10px;
    height: 24px;
  }

  .collapse-line-filed {
    /* margin-right: 10px; */
  }

  .collapse-line-other {
    display: flex;
    justify-content: flex-end;
  }

  .collapse-line-value {
    margin: 0 5px;
  }

  .collapse-delete {
    padding: 5px;
    height: 24px;
  }

  .sync {
    display: flex;
    align-items: center;
    font-size: 14px;
    margin-bottom: 10px;
    span {
      margin-right: 5px;
    }
  }

  .view-name-icon {
    position: relative;
    top: 2px;
  }

  .drag2 {
    display: flex;
    justify-content: center;
    align-items: center;

    .handle2 {
      position: relative;
      padding-top: 5px;
      margin-right: 10px;
    }
  }

  .collapse-btn {
    width: 65px;
  }

  .cursor-move {
    cursor: grab;
  }
</style>
