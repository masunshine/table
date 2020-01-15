<template>
  <div>
    <div class="userHead">
      <el-form ref="form" :model="params" label-width="auto" label-position="left">
        <el-row :gutter="20" type="flex" class="row-bg">
          <el-col :span="4" v-for="(item,index) in searchFrom" :key="index">
            <el-formItem :label="item.label">
              <el-input
                v-if="item.type === 'text'"
                v-model="params[item.prop]"
                clearable
                size="small"
              ></el-input>
              <el-select
                v-if="item.type === 'select'"
                v-model="params[item.prop]"
                clearable
                size="small"
              >
                <el-option v-for="(s,i) in item.data" :key="i" :label="s.label" :value="s.value"></el-option>
              </el-select>
              <select-from v-if="item.type === 'selectTips'" :dataSlpt="item" @blckData="blckData"></select-from>
            </el-formItem>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="getList">查询</el-button>
          </el-col>
        </el-row>
      </el-form>
    </div>
    <div class="userDesc">
      <el-button
        type="text"
        v-for="(btn,bIndex) in fromTool"
        :key="bIndex"
        :icon="btn.icon"
        :disabled="btnDisa(btn.disabled)"
        @click="btn.click"
        size="medium "
      >{{btn.label}}</el-button>
    </div>
    <div>
      <el-table border :highlight-current-row="true" :data="tableData">
        <el-table-column label="序号" width="50px" type="index"></el-table-column>
        <el-table-column
          v-for="(table,tabIndex) in tables.column"
          :key="tabIndex"
          :prop="table.prop"
          :label="table.label"
          :width="table.width"
          :sortable="table.sortable"
        ></el-table-column>
        <el-table-column
          v-if="tables.table"
          :fixed="tables.table.fixed"
          :label="tables.table.label"
          :width="tables.table.width"
        >
          <template slot-scope="scope">
            <el-button
              type="text"
              v-for="(btn,bIndex) in tables.table.btn"
              :key="bIndex"
              :icon="btn.icon"
              :disabled="btnDisa(btn.disabled)"
              @click="btn.click(scope.row)"
              size="medium "
            >{{btn.label}}</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="userPage">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>
  </div>
</template>

<script>
import selectFrom from "./from";
export default {
  name: "userManage",
  data() {
    return {
      tableData: [], // table列表数据
      totalData: [], // 后端不返回分页时 存入新的数组进行分页
      total: 0, // 总条数
      currentPage: 1, // 当前页
      pageSize: 10, // 每页多少条
      params: {}, // 头部from表单对象
      selectData: "", // 调用接口时select返回数据存储
      isPage: false // 判断是否后端支持分页
    };
  },
  created() {
    this.getList();
  },
  props: {
    searchFrom: {
      type: Array,
      default: () => []
    },
    fromTool: {
      type: Array,
      default: () => []
    },
    tables: {
      type: Object,
      default: () => null
    }
  },
  methods: {
    // 获取列表信息
    async getList() {
      if (this.isPage) {
        this.params.currentPage = this.currentPage;
        this.params.pageSize = this.pageSize;
      }
      let res = await this.postForm(this.tables.code, { ...this.params });
      if (res && res.data.code === "0") {
        // 查看返回数据是否有page，如果没有page组件自动做分页处理
        if (!res.data.page) {
          this.totalData = res.data.data;
          this.tableData = this.totalData.slice(0, 10);
          this.total = res.data.data.length;
          this.isPage = false;
        } else {
          this.tableData = res.data.data;
          this.total = res.data.page.total;
          this.isPage = true;
        }
      }
    },
    // 分页
    handleSizeChange(val) {
      this.pageSize = val;
      if (this.isPage) {
        this.getList();
      }
    },
    handleCurrentChange(val) {
      if (this.isPage) {
        this.currentPage = val;
        this.getList();
      } else {
        this.tableData = this.totalData.slice(10 * val - 10, 10 * val);
      }
    },
    // 下拉获取数据后返回的值
    blckData(val) {
      this.params[val.key] = val.value;
    },
    // 判断按钮是否隐藏 给一个默认值
    btnDisa(val) {
      if (!val) return false;
      return val;
    }
  },
  components: { selectFrom }
};
</script>

<style lang="scss" scope>
.userHead {
  border-bottom: 1px solid #f1f1f1;
  height: 50px;
  padding-top: 10px;
}
.userDesc {
  height: 40px;
  line-height: 40px;
  button {
    margin-right: 15px;
  }
}
.dialogSa {
  text-align: right;
  height: 40px;
  line-height: 50px;
  border-top: 1px solid #f1f1f1;
}
.userPage {
  margin-top: 15px;
  float: right;
}
</style>
