<!-- 通用Form -->
<template>
  <el-form
    v-if="showForm"
    ref="ProForm"
    :model="formParam"
    :label-position="labelPosition"
    :label-width="layout.labelWidth || '145px'"
    :rules="rules"
    :style="{ width: layout.formWidth || '560px' }"
    style="margin: 0 auto"
  >
    <el-row
      v-for="(row, j) in formRows"
      :key="j"
      :class="{ 'card-row': formRows.length > 1 }"
    >
      <el-col :span="24" class="card-title">
        <slot :name="'title' + j"></slot>
      </el-col>

      <slot v-if="j === 0" name="header"></slot>

      <el-row class="card-body" :gutter="layout.gutter || 20">
        <el-col
          v-for="(item, index) in row"
          :key="index"
          :span="item.form_span || 24"
          :xs="item.form_xs || item.form_span"
          :sm="item.form_sm || item.form_span"
          :md="item.form_md || item.form_span"
          :lg="item.form_lg || item.form_span"
          :xl="item.form_xl || item.form_span"
        >
          <el-form-item
            :prop="item.prop ? item.dataIndex : ''"
            :label="item.title + ' : '"
          >
            <el-input
              v-if="item.valueType === 'input'"
              v-model="formParam[item.dataIndex]"
              :type="item.inpuType || 'text'"
              :placeholder="item.placeholder || item.title"
            />
            <template v-else-if="item.valueType === 'check_code'">
              <el-row :gutter="16">
                <el-col class="gutter-row" :span="16">
                  <el-input v-model="formParam[item.dataIndex]" />
                </el-col>
                <el-col class="gutter-row" :span="8">
                  <SendCode :params="item.sendCode" />
                </el-col>
              </el-row>
            </template>
            <el-switch
              v-else-if="item.valueType === 'switch'"
              v-model="formParam[item.dataIndex]"
            />
            <el-date-picker
              v-else-if="item.valueType === 'date-picker'"
              v-model="formParam[item.dataIndex]"
              :type="item.pickerType"
              :format="item.pickerFormat"
              clearable
              style="width: 100%"
            />
            <el-select
              v-else-if="item.valueType === 'select'"
              v-model="formParam[item.dataIndex]"
              :placeholder="item.placeholder || item.title"
              filterable
              :multiple="item.multiple ? true : false"
              clearable
              style="width: 100%"
            >
              <el-option
                v-for="s in item.option.filter((i) => i.value !== '')"
                :key="'select' + s.value"
                :value="s.value"
                :label="s.label"
              >{{ s.label }}</el-option>
            </el-select>
            <el-checkbox-group
              v-else-if="item.valueType === 'checkbox'"
              v-model="formParam[item.dataIndex]"
            >
              <el-checkbox
                v-for="(s, i) in item.option"
                :key="i"
                :label="s.value"
              >{{ s.label }}</el-checkbox>
            </el-checkbox-group>
            <el-radio-group
              v-else-if="item.valueType === 'radio'"
              v-model="formParam[item.dataIndex]"
            >
              <el-radio
                v-for="(s, i) in item.option"
                :key="i"
                :label="s.value"
              >{{ s.label }}</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>

      <slot v-if="j === formRows.length - 1" name="footer"></slot>
    </el-row>

    <el-form-item
      v-if="!noFooter"
      label-width="0"
      style="margin-top: 24px; text-align: center"
    >
      <el-button
        :key="isEdit"
        size="large"
        type="primary"
        :disabled="btnDisabled"
        :loading="loading"
        @click="handleSubmit"
      >{{ isEdit ? '修改' : '确定' }}</el-button>
      <slot name="btn"></slot>
    </el-form-item>
  </el-form>
</template>

<script>
import SendCode from '@/components/sendCode/index.vue';
import { nextTick, ref, watch } from 'vue';
import Message from 'element-plus/lib/el-message';

export default {
  name: 'ProForm',
  components: { SendCode },
  props: {
    noFooter: {
      type: Boolean,
      default: false,
    },
    dialogVal: {
      type: Boolean,
      default: false,
    },
    formParam: {
      type: Object,
      default: () => {
        return {};
      },
    },
    layout: {
      type: Object,
      default: () => {
        return {};
      },
    },
    labelPosition: {
      type: String,
      default: 'right',
    },
    formList: {
      type: Array,
      default: () => {
        return [];
      },
    },
    isEdit: {
      type: Boolean,
      default: false,
    },
    btnDisabled: {
      type: Boolean,
      default: false,
    },
    subMet: {
      type: Function,
      default: () => {},
    },
    formCB: {
      type: Function,
      default: () => {},
    },
  },
  setup(prop) {
    const loading = ref(false);
    const showForm = ref(false);
    const rules = {};
    const formRows = ref([]);

    const ProForm = ref();
    const originalFormParams = JSON.parse(JSON.stringify(prop.formParam));

    function init() {
      for (let index = 0; index < prop.formList.length; index++) {
        const element = prop.formList[index];
        if (element.row || element.row === 0) {
          if (formRows.value[element.row]) {
            formRows.value[element.row].push(element);
          } else {
            formRows.value[element.row] = [];
            formRows.value[element.row].push(element);
          }
        }
        if (
          element.valueType &&
          element.valueType === 'select' &&
          element.optionMth
        ) {
          element.option = [];
          initOption(element);
        }
      }
      if (formRows.value.length === 0)
        formRows.value = [prop.formList.filter((i) => i.isForm)];
      initRules();
      showForm.value = true;
    }
    function initOption(element) {
      element.optionMth().then((res) => {
        if (!res) return;
        const arr = res.data.map((i) => {
          const obj = {};
          obj.label = i[element.optionskey.label];
          obj.value = i[element.optionskey.value];
          return obj;
        });
        element.defaultOption = element.defaultOption
          ? element.defaultOption
          : [];
        element.option = element.defaultOption.concat(arr);
      });
    }
    function initRules() {
      prop.formList.map((i) => {
        if (i.prop) {
          rules[i.dataIndex] = i.prop;
        }
      });
    }
    init();

    function resetFormParam() {
      Object.assign(prop.formParam, originalFormParams);
      ProForm.value.resetFields();
    }
    function handleSubmit() {
      ProForm.value.validate((valid) => {
        if (valid) {
          loading.value = true;
          prop
            .subMet()
            .then((res) => {
              Message({ message: res.msg, type: 'success' });
              prop.formCB();
              resetFormParam();
            })
            .finally(() => {
              loading.value = false;
            });
        }
      });
    }

    watch(
      () => prop.dialogVal,
      (val) => {
        if (val) {
          nextTick().then(() => {
            ProForm.value.clearValidate();
          });
        }
      }
    );

    return {
      ProForm,
      loading,
      formRows,
      showForm,
      rules,
      handleSubmit,
    };
  },
};
</script>
<style lang="scss" scoped>
.card-row {
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  border-radius: 4px;
  border: 1px solid #ebeef5;
  background-color: #fff;
  overflow: hidden;
  color: #303133;
  transition: 0.3s;
  .card-title {
    padding: 18px 20px;
    border-bottom: 1px solid #ebeef5;
    box-sizing: border-box;
  }
  .card-body {
    padding: 20px;
  }
}
.card-row + .card-row {
  margin-top: 20px;
}
</style>
