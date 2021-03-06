<template>
  <el-select
    ref="wl-vue-select"
    :value="value"
    :size="size"
    :multiple="multiple"
    :disabled="disabled"
    :value-key="valueKey"
    :clearable="clearable"
    :filterable="filterable"
    :placeholder="placeholder"
    :allowCreate="allowCreate"
    :multipleLimit="multipleLimit"
    @change="selectChange"
  >
    <el-option
      label="全部"
      key="select-all"
      :value="
        valueKey
          ? { [selfProps.value]: empty, [selfProps.label]: '全部' }
          : empty
      "
      v-if="multiple && selfData.length > showTotal"
    ></el-option>
    <el-option
      v-for="item in selfData"
      :key="item[selfProps.value]"
      :label="item[selfProps.label]"
      :value="valueKey ? item : item[selfProps.value]"
    ></el-option>
  </el-select>
</template>

<script>
/**
 * auth:weilan
 * github: https://github.com/hql7
 * description: 基于el-select的带全选功能的下拉框组件
 * props:输入值
 *    data->可选项数据
 *    props->配置项 label:"label", value: "value"
 *    showTotal-> 多选时,当可选项大于多少个时显示【全部】字段。（需配合multiple使用）
 *    value-> v-model绑定值
 *    others->见https://element.eleme.cn/#/zh-CN/component/select
 * emit:输出
 */
export default {
  name: "wlVueSelect",
  data() {
    return {
      // value: "", // 当前选中值
      empty: "00000000-0000-0000-0000-000000000000" // 全部时的空id
    };
  },
  model: {
    prop: "value", //这里使我们定义的v-model属性
    event: "change"
  },
  props: {
    // 数据
    data: {
      type: Array,
      default: () => {
        return [];
      }
    },
    // 配置项 label:"label", value: "value"
    props: Object,
    // 多选时,当可选项大于多少个时显示【全部】字段
    showTotal: {
      type: Number,
      default: 1
    },
    // 是否默认选中，如果开启有全部时选中全部，无全部时选中第一个。(开启此功能请不要给v-model赋初始值)
    defaultSelect: {
      type: Boolean,
      default: false
    },
    // v-model绑定值
    value: [String, Array, Object],
    // value-key 作为 value 唯一标识的键名，如传则认为绑定值为对象
    valueKey: [String],
    // 是否多选
    multiple: {
      type: Boolean,
      default: false
    },
    disabled: {
      // 是否禁用
      type: Boolean,
      default: false
    },
    size: String, // 输入框尺寸medium/small/mini
    clearable: {
      // 是否可以清空选项
      type: Boolean,
      default: false
    },
    multipleLimit: {
      // 多选时用户最多可以选择的项目数，为 0 则不限制
      type: Number,
      default: 0
    },
    placeholder: {
      // 占位符
      type: String,
      default: "请选择"
    },
    filterable: {
      // 是否可搜索
      type: Boolean,
      default: false
    },
    allowCreate: {
      // 是否允许用户创建新条目，需配合 filterable 使用
      type: Boolean,
      default: false
    },
    // 多选时，清空选项关闭
    noCheckedClose: {
      type: Boolean,
      default: false
    },
  },
  methods: {
    selectChange(val) {
      if (
        !this.multiple ||
        val.length === 0 ||
        this.showTotal >= this.selfData.length
      ) {
        this.$emit("change", val);
        if (val.length == 0 && this.noCheckedClose)
          this.$refs["wl-vue-select"].blur();
        return;
      }
      // 处理多选的全选效果
      let _data = [];
      let _all_item_index = this.valueKey
        ? val.findIndex(item => item[this.selfProps.value] === this.empty)
        : val.findIndex(item => item === this.empty);
      if (_all_item_index === -1 && val.length === this.selfData.length) {
        _data = this.valueKey
          ? [
              {
                [this.selfProps.value]: this.empty,
                [this.selfProps.label]: "全部"
              }
            ]
          : [this.empty];
      } else {
        let arr_length_index = val.length - 1;
        if (_all_item_index === arr_length_index) {
          _data = [val[arr_length_index]];
        } else {
          _data = this.valueKey
            ? val.filter(item => item[this.selfProps.value] !== this.empty)
            : val.filter(item => item !== this.empty);
        }
      }
      /* let _all_item = this.valueKey
        ? val.find(item => item[this.selfProps.value] === this.empty)
        : val.find(item => item === this.empty);
      let _data = _all_item
        ? [_all_item]
        : val.length !== this.data.length
        ? val
        : this.valueKey
        ? [
            {
              [this.selfProps.value]: this.empty,
              [this.selfProps.label]: "全部"
            }
          ]
        : [this.empty]; */
      this.$emit("change", _data);
    }
  },
  computed: {
    // 动态数据
    selfData() {
      return this.data;
    },
    // props处理
    selfProps() {
      return { label: "label", value: "value", ...this.props };
    }
  },
  watch: {
    // 处理默认选中
    selfData(val) {
      if (!this.defaultSelect || val.length === 0) return;
      if (!this.multiple) {
        this.$emit("change", val[0]);
      } else {
        let select_item =
          val.length > this.showTotal
            ? {
                [this.selfProps.value]: this.empty,
                [this.selfProps.label]: "全部"
              }
            : val[0];
        let _data = !this.valueKey
          ? select_item[this.selfProps.value]
          : select_item;
        this.$emit("change", [_data]);
      }
    }
  }
};
</script>
