<template>
  <div class="crud-container pull-auto">
    <div class="crud-header">
      <el-collapse-transition>
        <el-form :model="searchForm"
                 :inline="true"
                 ref="searchForm"
                 v-if="searchFlag && searchShow ">
          <!-- 循环列搜索框 -->
          <el-form-item :label="column.label"
                        :prop="column.prop"
                        v-for="(column,index) in option.column"
                        :key="index"
                        v-if="column.search">
            <component :size="option.searchSize"
                       :is="getSearchType(column.type)"
                       v-model="searchForm[column.prop]"
                       :type="column.type"
                       clearable:type="column.type"
                       :placeholder="column.label"
                       :dic="setDic(column.dicData,DIC[column.dicData])"></component>
          </el-form-item>
          <el-form-item>
            <el-button type="primary"
                       @click="searchChnage"
                       icon="el-icon-search"
                       :size="option.searchSize">搜索</el-button>
            <el-button @click="searchReset"
                       icon="el-icon-delete"
                       :size="option.searchSize">清空</el-button>
          </el-form-item>
        </el-form>
      </el-collapse-transition>
    </div>
    <!-- 表格功能列 -->
    <div class="crud-menu">
      <div class="crud-menu_left">
        <el-button type="primary"
                   @click="rowAdd"
                   icon="el-icon-plus"
                   v-if="vaildData(option.addBtn,true)"
                   size="small">新 增</el-button>
        <slot name="menuLeft"></slot>
      </div>
      <div class="crud-menu_right">
        <slot name="menuRight"></slot>
        <el-button icon="el-icon-refresh"
                   circle
                   size="small"
                   @click="refreshChange"
                   v-if="vaildData(option.refreshBtn,true)"></el-button>
        <el-button icon="el-icon-menu"
                   circle
                   size="small"
                   @click="showClomnuBox=true"
                   v-if="vaildData(option.showClomnuBtn,true)"></el-button>
        <el-button icon="el-icon-search"
                   circle
                   size="small"
                   @click="searchShow=!searchShow"
                   v-if="vaildData(option.showSearchBtn,true) && searchFlag"></el-button>
      </div>
    </div>
    <!-- <el-tag class="avue-tip"
            v-if="vaildData(option.tip,true) && option.selection">
      <i class="el-icon-info">&nbsp;</i>
      <span class="name">当前表格已选择
        <span class="count">{{selectLen}}</span> 项</span>
      <span class="menu">
        <span @click="selectClear"
              v-if="vaildData(option.selectClearBtn,true) && option.selection">清空</span>
      </span>
    </el-tag> -->
    <el-table :data="data"
              :stripe="option.stripe"
              :show-header="option.showHeader"
              :default-sort="option.defaultSort"
              @row-click="rowClick"
              @row-dblclick="rowDblclick"
              :row-class-name="rowClassName"
              max-height="option.maxHeight"
              :height="option.height=='auto'?($AVUE.clientHeight - vaildData(option.calcHeight,320)):option.height"
              ref="table"
              :width="setPx(option.width,'100%')"
              :border="option.border"
              v-loading="tableLoading"
              @selection-change="selectionChange"
              @sort-change="sortChange">
      <!-- 下拉弹出框  -->
      <template v-if="option.expand">
        <el-table-column type="expand"
                         width="50"
                         fixed="left"
                         align="center">
          <template slot-scope="props">
            <slot :row="props.row"
                  name="expand"></slot>
          </template>
        </el-table-column>
      </template>
      <!-- 选择框 -->
      <template v-if="option.selection">
        <el-table-column type="selection"
                         width="50"
                         fixed="left"
                         align="center">
        </el-table-column>
      </template>

      <!-- 序号 -->
      <template v-if="option.index">
        <el-table-column :label="vaildData(option.indexLabel,'#')"
                         type="index"
                         width="50"
                         :index="indexMethod"
                         fixed="left"
                         align="center">
        </el-table-column>
      </template>
      <!-- 循环列 -->
      <el-table-column v-if="showClomnuIndex.indexOf(index)!=-1"
                       v-for="(column,index) in option.column"
                       :prop="column.prop"
                       :key="column.prop"
                       :show-overflow-tooltip="column.overHidden"
                       :min-width="column.minWidth"
                       :sortable="column.sortable"
                       :align="vaildData(column.align,option.align)"
                       :header-align="vaildData(column.headerAlign,option.headerAlign)"
                       :width="column.width"
                       :label="column.label"
                       :fixed="column.fixed">
        <template slot-scope="scope">
          <slot :row="scope.row"
                :dic="setDic(column.dicData,DIC[column.dicData])"
                :label="findByvalue(setDic(column.dicData,DIC[column.dicData]),scope.row[column.prop],(column.props || option.props))"
                :name="column.prop"
                v-if="column.solt"></slot>
          <template v-else>
            <span v-html="detail(scope.row,column)"
                  v-if="column.type"></span>
            <span v-else>{{scope.row[column.prop]}}</span>
          </template>
        </template>
      </el-table-column>
      <el-table-column fixed="right"
                       v-if="vaildData(option.menu,true)"
                       label="操作"
                       :align="option.menuAlign"
                       :header-align="option.menuHeaderAlign"
                       :width="vaildData(option.menuWidth,240)">
        <template slot-scope="scope">
          <template v-if="vaildData(option.menu,true)">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="small"
                       @click.stop.safe="rowEdit(scope.row,scope.$index)"
                       v-if="vaildData(option.editBtn,true)">编 辑</el-button>
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="small"
                       @click.stop.safe="rowDel(scope.row,scope.$index)"
                       v-if="vaildData(option.delBtn,true)">删 除</el-button>
          </template>
          <slot :row="scope.row"
                name="menu"
                :index="scope.$index"></slot>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination v-if="vaildData(option.page,true)"
                   class="crud-pagination pull-right"
                   :current-page.sync="page.currentPage"
                   :background="vaildData(option.pageBackground,true)"
                   :page-size="page.pageSize"
                   :page-sizes="page.pageSizes"
                   @size-change="sizeChange"
                   @current-change="currentChange"
                   layout="total, sizes, prev, pager, next, jumper"
                   :total="page.total"></el-pagination>
    <!-- 表单 -->
    <el-dialog lock-scroll
               :custom-class="vaildData(option.customClass,'')"
               :fullscreen="vaildData(option.formFullscreen,false)"
               :modal-append-to-body="false"
               :append-to-body="true"
               :title="boxType=='add'?'新增':'编辑'"
               :visible.sync="boxVisible"
               :width="vaildData(option.formWidth,'50%')"
               @close="hide">
      <div class="avue-dialog">
        <avue-form v-model="tableForm"
                   ref="tableForm"
                   :option="formOption">
          <template slot-scope="scope"
                    v-for="(item,index) in option.column"
                    :slot="item.prop">
            <slot :value="scope.value"
                  :column="scope.column"
                  :dic="scope.dic"
                  :name="item.prop+'Form'"
                  v-if="item.formsolt"></slot>
          </template>
        </avue-form>
      </div>
      <span slot="footer"
            class="dialog-footer">
        <slot name="menuForm"
              :row="tableForm"
              :type="boxType"></slot>
        <el-button type="primary"
                   @click="rowUpdate"
                   v-if="boxType=='edit'">修 改</el-button>
        <el-button type="primary"
                   @click="rowSave"
                   v-else-if="boxType=='add'">新 增</el-button>
        <el-button @click="hide">取 消</el-button>
      </span>
    </el-dialog>
    <!-- 动态列 -->
    <el-dialog lock-scroll
               :modal-append-to-body="false"
               :append-to-body="true"
               title="多选"
               :visible.sync="showClomnuBox">
      <el-checkbox-group v-model="showClomnuIndex">
        <el-row :span="24">
          <el-col :span="6"
                  v-for="(item,index) in showClomnuList"
                  :key="index">
            <el-checkbox :label="item.index">{{item.label}}</el-checkbox>
          </el-col>
        </el-row>
      </el-checkbox-group>
    </el-dialog>

  </div>
</template>
<script>
import crud from "../../mixins/crud.js";
import { validatenull } from "../../utils/validate.js";
import moment from "moment";
export default {
  name: "AvueCrud",
  mixins: [crud()],
  components: {},
  data () {
    return {
      defaultForm: {
        tableForm: {},
        searchForm: {}
      },
      searchShow: false,
      searchForm: {},
      boxVisible: false,
      boxType: 'add',
      showClomnuIndex: [],
      showClomnuBox: false,
      showClomnuList: [],
      tableForm: {},
      tableFormRules: {},
      tableIndex: -1,
      tableSelect: []
    };
  },
  created () {
    //初始化动态列
    this.showClomnuInit();
  },
  computed: {
    selectLen () {
      return this.tableSelect ? this.tableSelect.length : 0;
    },
    searchFlag: function () {
      if (validatenull(this.searchForm)) {
        this.searchShow = false;
        return false;
      } else {
        this.searchShow = true;
        return true
      }
    },
    formOption: function () {
      let option = Object.assign({}, this.option);
      option.submitBtn = false;
      option.submitPostion = 'right';
      option.boxType = this.boxType;
      return option;
    }
  },
  mounted () { },
  props: {
    value: {
      type: Object,
      default: () => {
        return {}
      }
    },
    beforeClose: Function,
    beforeOpen: Function,
    rowClassName: Function,
    page: {
      type: Object,
      default () {
        return {
          total: 0, //总页数
          currentPage: 0, //当前页数
          pageSize: 10, //每页显示多少条
          pageSizes: [10, 20, 30, 40, 50, 100],
          background: true //背景颜色
        };
      }
    },
    tableLoading: {
      type: Boolean,
      default: false
    },
    data: {
      type: Array,
      required: true,
      default: () => {
        return [];
      }
    },
    option: {
      type: Object,
      required: true,
      default: () => {
        return [];
      }
    }
  },
  methods: {
    selectClear () {
      this.$refs.table.clearSelection();
    },
    indexMethod (index) {
      return (index + 1) + (((this.page.currentPage || 1) - 1) * (this.page.pageSize || 10))
    },
    showClomnu () { },
    refreshChange () {
      this.$emit("refresh-change", this.page);
    },
    rulesInit () {
      this.tableFormRules = {};
      this.option.column.forEach(ele => {
        if (ele.rules) this.tableFormRules[ele.prop] = ele.rules;
      });
    },
    showClomnuInit: function () {
      this.option.column.forEach((ele, index) => {
        if (validatenull(ele.hide)) {
          this.showClomnuIndex.push(index);
        }
        if (ele.showClomnu != false) {

          let obj = {
            label: ele.label,
            index: index
          };
          this.showClomnuList.push(Object.assign({}, obj));
        }
      });
    },
    formVal () {
      for (let o in this.value) {
        this.tableForm[o] = this.value[o];
      }
      this.$emit("input", this.tableForm);
    },
    formInit () {
      this.defaultForm = this.formInitVal(this.option.column);
      this.tableForm = Object.assign({}, this.defaultForm.tableForm);
      this.searchForm = Object.assign({}, this.defaultForm.searchForm);
      this.formVal();
    },
    //搜索清空
    searchReset () {
      this.$refs["searchForm"].resetFields();
    },
    // 页大小回调
    sizeChange (val) {
      this.$emit("size-change", val);
    },
    // 页码回调
    currentChange (val) {
      this.$emit("current-change", val);
    },
    // 选中实例
    toggleSelection (rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.table.toggleRowSelection(row);
        });
      } else {
        this.$refs.table.clearSelection();
      }
    },
    //选择回调
    selectionChange (val) {
      this.tableSelect = val;
      this.$emit("selection-change", val);
    },
    //排序回调
    sortChange (val) {
      this.$emit("sort-change", val);
    },
    //搜索回调
    searchChnage () {
      this.$emit("search-change", this.searchForm);
    },
    //行双击
    rowDblclick (row, event) {
      this.$emit("row-dblclick", row, event);
    },

    //行单机
    rowClick (row, event, column) {
      this.$emit("row-click", row, event, column);
    },
    //处理数据
    detail (row, column) {
      let result = row[column.prop] || "";
      if (column.type) {
        if (
          (column.type == "date" ||
            column.type == "time" ||
            column.type == "datetime") &&
          column.format
        ) {
          const format = column.format
            .replace("dd", "DD")
            .replace("yyyy", "YYYY");
          result = moment(result).format(format);
        }
        if (column.dicData) {
          result = this.findByvalue(
            typeof column.dicData == "string"
              ? this.DIC[column.dicData]
              : column.dicData,
            result
          );
        }
      }
      if (column.formatter && typeof column.formatter === "function") {
        result = column.formatter(row, result);
      }
      return result;
    },
    // 新增
    rowAdd () {
      this.boxType = 'add';
      this.tableForm = Object.assign({}, this.defaultForm.tableForm);
      this.$emit("input", this.tableForm);
      this.show();
    },
    // 编辑
    rowEdit (row, index) {
      this.tableForm = Object.assign({}, row);
      this.$emit("input", this.tableForm);
      this.tableIndex = index;
      this.boxType = 'edit';
      this.show();
    },
    // 删除
    rowDel (row, index) {
      this.$emit("row-del", row, index);
    },
    //保存
    rowSave () {
      this.$refs["tableForm"].validate().then(res => {
        this.$emit("row-save", Object.assign({}, this.tableForm), this.hide);
      });
    },
    //更新
    rowUpdate () {
      this.$refs["tableForm"].validate().then(res => {
        const index = this.tableIndex;
        this.$emit(
          "row-update",
          Object.assign({}, this.tableForm),
          index,
          this.hide
        );
      })
    },
    //显示表单
    show (cancel) {
      const callack = () => {
        if (cancel !== true) {
          this.$nextTick(() => {
            this.$refs["tableForm"].clearValidate();
          });
          this.boxVisible = true;
        }
      };
      if (typeof this.beforeOpen === "function") this.beforeOpen(callack);
      else callack();
    },
    //隐藏表单
    hide (cancel) {
      const callack = () => {
        if (cancel !== false) {
          this.boxVisible = false;
          this.$nextTick(() => {
            this.$refs["tableForm"].resetForm();
            this.$emit("input", this.tableForm);
          });
        }
      };
      if (typeof this.beforeClose === "function") this.beforeClose(callack);
      else callack();
    }
  }
};
</script>

<style lang="scss">
.crud-container {
  margin: 0 auto;
  width: 99%;
  .el-table__header th {
    word-break: break-word;
    color: rgba(0, 0, 0, 0.85);
    background: #fafafa;
  }
}
.crud-pagination {
  margin-top: 15px;
  padding: 10px 20px;
}
.crud-form {
  padding: 0 8px;
}
.crud-header {
  margin-bottom: 10px;
  & > .el-button {
    padding: 12px 25px;
  }
}

.crud-menu {
  position: relative;
  width: 100%;
  min-height: 40px;
  height: auto;
  overflow: hidden;
  margin-bottom: 12px;
  .crud-menu_left,
  .crud-menu_right {
    position: absolute;
    height: auto;
    overflow: hidden;
  }
  .crud-menu_left {
    left: 0;
    .el-button {
    }
  }
  .crud-menu_right {
    right: 0;
    .el-button {
      margin-left: 5px;
    }
  }
}
.crud-dialog {
  border-radius: 5px;
  box-shadow: 10px 10px 15px #666;
  .el-dialog__header {
    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
    padding: 12px 20px;
    background-color: #409eff;
  }
  .el-dialog__close {
    color: #fff;
    font-size: 18px;
    &:hover {
      color: #ffddff;
    }
  }
  .el-dialog__title {
    color: #fff;
    font-size: 18px;
  }
  .el-dialog__headerbtn {
    top: 18px;
  }
}
.crud--overflow {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  display: block;
}
</style>
