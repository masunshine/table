<template>
  <div>
    <el-select v-model="params[dataSlpt.prop]" clearable size="small" @change="commonSelect">
      <el-option v-for="(f,fi) in fromList" :key="fi" :label="f.label" :value="f.value"></el-option>
    </el-select>
  </div>
</template>

<script>
export default {
  data() {
    return {
      params: {},
      fromList: []
    };
  },
  created() {
    this.codeSelect(this.dataSlpt.code, this.dataSlpt.prop);
  },
  props: {
    dataSlpt: {
      type: Object,
      default: () => null
    }
  },
  methods: {
    // 头部筛选列表通过code获取信息
    async codeSelect(code, prop) {
      let res = await this.postForm(code, { ...this.params });
      if (res) {
        this.fromList = res;
      }
    },
    // 获取选中的值返回给父级组件
    commonSelect() {
      let data = {
        key: this.dataSlpt.prop,
        value: this.params[this.dataSlpt.prop]
      };
      this.$emit("blckData", data);
    }
  },
  components: {}
};
</script>
