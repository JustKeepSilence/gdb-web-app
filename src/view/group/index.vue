<template>
  <div>
    <!-- 头部搜索框区域 -->
    <el-row :gutter="3" style="margin-top: -10px">
      <el-col :span="2" :xs="8">
        <el-select
          :size="size"
          v-model="selectedGroups"
          placeholder="请选择分组"
          @change="getSelectedGroupInfos"
        >
          <el-option
            v-for="(item, index) in groups"
            :key="index"
            :label="item"
            :value="item"
          ></el-option>
        </el-select>
      </el-col>
      <el-col :span="2" :xs="6">
        <el-button type="success" class="hidden-xs-only" icon="el-icon-plus" @click="openGroupDialog"
          >批量加组</el-button
        >
      </el-col>
      <el-col :span="2" :xs="8">
        <el-button type="success"  icon="el-icon-plus" @click="openItemDialog"
          >单个加点</el-button
        >
      </el-col>
      <el-col :span="2" :xs="6">
        <el-button
          type="success"
          class="hidden-xs-only"
          icon="el-icon-folder-opened"
          @click="openItemsDialog"
          >批量加点</el-button
        >
      </el-col>
      <el-col :span="2" :xs="8">
        <el-button
          type="primary"
          icon="el-icon-download"
          @click="handleItemsDownload"
          >点表下载</el-button
        >
      </el-col>
      <el-col :span="2" :xs="6">
        <el-button type="primary" class="hidden-xs-only" icon="el-icon-edit" @click="handleEditGroup" >编辑此组</el-button>
      </el-col>
      <el-col :span="2" :xs="6">
        <el-button type="danger" class="hidden-xs-only" icon="el-icon-delete" @click="handleGroup" :disabled="handleDeleteGroupButtonState"
          >删除此组</el-button
        >
      </el-col>
      <el-col :span="4" :xs="6">
        <el-input
          class="hidden-xs-only"
          prefix-icon="el-icon-search"
          v-model="searchKeyWord"
          placeholder="输入关键字搜索"
        />
      </el-col>
      <el-col :span="2" :xs="6">
        <el-button type="primary" class="hidden-xs-only" @click="handleSearch" icon="el-icon-search"
          >表格搜索</el-button
        >
      </el-col>
      <el-col :span="2" :xs="6">
        <el-button type="danger" class="hidden-xs-only" @click="handleDeleteHistory" icon="el-icon-delete"
          >历史删除</el-button
        >
      </el-col>
    </el-row>
    <el-divider></el-divider>
    <!-- table区域 -->
    <el-table
      :data="itemData"
      :size="size"
      :max-height="maxHeight"
      border
       @selection-change="handleSelection"
      style="width: 100%; margin-top: -15px"
      v-loading="tableLoading"
      element-loading-background="transparent"
      :element-loading-text="tableLoadingText"
      row-key="index"
    >
      <el-table-column
        v-for="(item, index) in tableColumns"
        v-if="index === 0"
        :key="Math.random()"
        :prop="item.prop"
        :width="indexWidth"
        :reserve-selection="true"
        type="selection"
        fixed="left"
      ></el-table-column>
      <el-table-column
        v-else-if="index === tableColumns.length - 1"
        fixed="right"
        align="center"
        label="操作"
        :key="Math.random()"
        :width="opWidth"
      >
        <template slot-scope="scope">
          <el-button @click="handleHisroty(scope.row)" type="text"
            >查看</el-button
          >
          <el-button type="text" @click="editItem(scope.row)">编辑</el-button>
          <el-button type="text" @click="deleteItem(scope.row)">删除</el-button>
        </template>
      </el-table-column>
      <el-table-column
        v-else
        show-overflow-tooltip
        :prop="item.prop"
        align="center"
        :key="Math.random()"
        :label="item.label"
        :min-width="item.width"
      >
      </el-table-column>
    </el-table>
    <!-- 分页区域 -->
    <el-row style="margin-top: 10px">
      <el-pagination
       v-if='!small'
        background
        layout="total, sizes, prev, pager, next, jumper"
        :total="itemCount"
        :current-page.sync="currentItemPage"
        @size-change="handleSizeChange"
        @current-change="handlePageChange"
        :page-sizes="[10, 15, 20]"
        :page-size="rowCount"
      ></el-pagination>
      <el-pagination v-else :total="itemCount"
        small
        layout="prev, pager, next"
        :current-page.sync="currentItemPage"
        @size-change="handleSizeChange"
        @current-change="handlePageChange"
        :page-size="rowCount"
        >  
      </el-pagination>
    </el-row>
    <!-- 加组弹窗 -->
    <el-dialog
      :title="groupDialogName"
      :visible.sync="groupDialog"
      width="800px"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-form ref="groupForm" :model="groupForm" label-width="100px">
        <el-form-item label="GroupName">
          <el-input
            v-model="groupForm.groupName"
            placeholder="组名请以英文逗号,分隔"
          ></el-input>
        </el-form-item>
        <el-form-item label="ColumnName">
          <el-input
            v-model="groupForm.columnName"
            placeholder="列名请以英文逗号,分隔,组之间请以|分割"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="handleAddGroups">确定</el-button>
        <el-button @click="groupDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 单个加点的弹窗 -->
    <el-dialog
      :title="itemDialogName"
      :visible.sync="itemDialog"
      :width="width"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-form label-width="100px">
        <el-form-item
          v-for="(item, index) in groupColumns"
          :label="item.label"
          :key="index"
        >
          <el-input v-model="item.model" ref="itemForm" :size="size"> </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="handleAddItem">确定</el-button>
        <el-button @click="itemDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 批量加点弹窗 -->
    <el-dialog
      :title="itemDialogNames"
      :visible.sync="itemsDialog"
      v-loading="addItemLoading"
      element-loading-background="transparent"
      element-loading-text="拼命加点中..."
      width="800px"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-upload
        accept=".xls,.xlsx"
        drag
        :action="actionUrl"
        multiple
        :headers="uploadHeaders"
        :http-request="uploadFile"
        :before-upload="beforeUpload"
        :limit="limit"
        :file-list="fileList"
        :on-exceed="showExceed"
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        <div class="el-upload__tip" slot="tip">可以上传xlsx或者xls文件</div>
      </el-upload>
      <div slot="footer" class="dialog-footer">
        <el-button @click="handleAddItems">确定</el-button>
        <el-button @click="itemsDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 历史曲线 -->
    <el-dialog
      :title="historyDialogName"
      :visible="historyDialog"
      :width="widthChart"
      v-loading="chartLoading"
      element-loading-background="transparent"
      element-loading-text="数据下载中..."
      :showClose="false"
      :body="showChartBody"
      @opened="handleOpen"
    >
      <div class="block">
        <el-row>
          <el-col :span="14" :xs="24" class="hidden-xs-only">
            <span style="margin-left: 20px"
              >请选择时间</span
            >
            <el-date-picker
              :size="size"
              v-model="st"
              type="datetimerange"
              :picker-options="pickerOptions"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              align="right"
              @change="showHistory"
            >
            </el-date-picker>
          </el-col>
          <el-col :span="8" :xs="16" class="hidden-xs-only">
            <el-form ref="form" label-width="80px">
              <el-form-item label="取数间隔" style="margin-bottom: 0px">
                <el-select v-model="interval" placeholder="请选择" @change="showHistory" :size="size">
                  <el-option
                    v-for="(item, index) in intervals"
                    :key="index"
                    :label="item.label"
                    :value="item.value"
                  >
                  </el-option>
                </el-select>
              </el-form-item>
            </el-form>
          </el-col>
        </el-row>
      </div>
      <el-divider></el-divider>
      <div id="main" :width="chartContainerWidth" style="height: 400px"></div>
      <div slot="footer" class="dialog-footer">
        <el-button　@click="handleDownloadData">数据下载</el-button>
        <el-button @click="historyDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 编辑此组的弹窗 -->
    <el-dialog
      :title="editGroupDialogNames"
      :visible.sync="editGroupDialog"
      width="550px"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-table
    :data="groupTableData"
    style="width: 100%"
    max-height="250">
    <el-table-column
      prop="columnName"
      align="center"
      label="columnName"
      width="320">
    </el-table-column>
    <el-table-column
      align="center"
      fixed="right"
      label="操作"
      width="120">
      <template slot-scope="scope">
        <el-button
          @click="editColumns(scope.row)"
          type="text"
          size="small">
          编辑
        </el-button>
        <el-button
          @click="deleteColumns(scope.row)"
          type="text"
          size="small">
          删除
        </el-button>
      </template>
    </el-table-column>
  </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addColumns">增加列</el-button>
        <el-button @click="editGroupDialogCloseHandler">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 编辑列名弹窗 -->
    <el-dialog
      :title="editColumnDialogNames"
      :visible.sync="editColumnDialog"
      width="550px"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-input v-model="editedColumnName"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editColumnHandler">确定</el-button>
        <el-button @click="editColumnDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 增加列的弹窗 -->
    <el-dialog
      :title="addColumnDialogNames"
      :visible.sync="addColumnDialog"
      width="550px"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-input v-model="addedColumnName" placeholder="不同的列名之间请以英文逗号隔开"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addColumnHandler">确定</el-button>
        <el-button @click="addColumnDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 编辑item的弹窗 -->
    <el-dialog
      :title="editItemDialogName"
      :visible.sync="editItemDialog"
      :width="width"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-form label-width="100px">
        <el-form-item
          v-for="(item, index) in editItems"
          :label="item.label"
          :key="index"
        >
          <el-input v-model="item.value" :size="size"> </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editItemsHandler">确定</el-button>
        <el-button @click="editItemDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 删除历史弹窗 -->
    <el-dialog
      :title="deleteHistoryDialogName"
      :visible="deleteHistoryDialog"
      :width="widthChart"
      @opened="changeTransferWidth"
    >
      <div class="block">
        <el-row>
          <el-col :span="14" :xs="24" class="hidden-xs-only">
            <span style="margin-left: 20px"
              >请选择时间</span
            >
            <el-date-picker
              :size="size"
              v-model="dst"
              type="datetimerange"
              :picker-options="pickerOptions"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              align="right"
            >
            </el-date-picker>
          </el-col>
        </el-row>
      </div>
      <el-divider></el-divider>
      <el-transfer v-model="transferValue" :data="transferData" :titles="titles" @change="updateDeletedRows"></el-transfer>
      <div slot="footer" class="dialog-footer">
        <el-button @click="deleteHistory">删除历史</el-button>
        <el-button @click="deleteHistoryDialog = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 删除历史item信息确认弹窗 -->
    <el-dialog
      title="信息确认"
      :visible.sync="infoConfirmnDialog"
      width="600px"
      v-loading="deleteHistoryLoading"
      element-loading-text="删除历史中"
      element-loading-background="transparent"
      :showClose="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <h3>要删除历史的Item(时间{{timeStamp}})</h3>
      <div class="demo-block" style="overflow:auto">
        <div class="source">
          <div>
            <ul class="infinite-list" v-infinite-scroll="loadItems" style="overflow:auto">
              <li v-for="(item,index) in itemInfos" :key="index" class="infinite-list-item">{{ item.itemName }}</li>
            </ul>
      </div>
      </div>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="confirmDeleteHistory">确定</el-button>
        <el-button @click="infoConfirmnDialog = false">关闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import {
  getGroups,
  addGroups,
  getGroupColumns,
  addItem,
  deleteGroup,
  getItems,
  getRealTimeData,
  addItemsByExcel,
  getHistoryData,
  updateColumnNames,
  addColumns,
  deleteColumns,
  updateItems,
  deleteItems,
  addCalulationItems,
  deleteHistory,
} from "@/api/group"; // 导入请求的接口
import { exportDataToExcel } from "@/utils/excel";
import axios from "axios";
const excelJs = require("exceljs"); // 导入exceljs
import { saveAs } from "file-saver";
import { getCookie } from "@/utils/cookie";
import { quillEditor } from "vue-quill-editor";
import * as Quill from "quill";
import "element-ui/lib/theme-chalk/display.css";
const _ = require("lodash");

export default {
  name: "Group",
  data: () => {
    return {
      selectedGroups: "", // 被选中的分组
      groups: [], // gdb中已经存在组
      groupDialog: false, // 加组弹窗
      groupDialogName: "批量加组",
      groupForm: {
        groupName: "",
        columnName: "",
      },
      groupColumns: [], // 选中组的列名
      itemDialog: false, // 单个加点的弹窗
      itemDialogName: "单个加点",
      itemData: [], // 表中的数据
      tableColumns: [], // 表头
      indexWidth: "60", // Index的宽度
      opWidth: "200", // 操作列的宽度
      searchKeyWord: "", // 表格搜索关键字
      itemDialogNames: "批量加点", // title
      itemsDialog: false,
      actionUrl: "", // 上传文件的url
      uploadHeaders: { "Content-Type": "multipart/form-data" }, // 上传的头部
      fileList: [], // 上传的文件列表
      fileContent: null,
      limit: 1, // 文件限制
      startRow: 0, // 开始的row
      rowCount: 10, // 每页的数据
      currentItemPage: 1, // 当前的页码
      itemCount: 0, // 总条数
      addItemLoading: false, // 加点loading
      searchCondition: "itemName like '%%'", // 根据searchKeyWord组成的搜索条件
      tableLoading: false, // 加载table
      tableLoadingText: "表格数据加载中...",
      historyDialogName: "历史曲线查看",
      historyDialog: false,
      myChart: null,
      st: [
        new Date(new Date().setTime(new Date() - 3600 * 1000 * 24)),
        new Date(),
      ], // 时间选择器的时间
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一小时",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 1);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近一天",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 1);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            },
          },
        ],
      },
      intervals: [
        { label: "1s", value: "1" },
        { label: "10s", value: "10" },
        { label: "30s", value: "30" },
        { label: "60s", value: "60" },
        { label: "120s", value: "120" },
        { label: "360s", value: "360" },
      ], // 取数间隔选择
      interval: "60", // 取数间隔
      selectedItem: "", // 选择的item
      chartItemValues: null, // chart中的数据
      chartLoading: false,
      showChartBody: document.getElementById("main"),
      handleDeleteGroupButtonState: null, // 编辑按钮的状态
      editGroupDialogNames: "编辑此组",
      editGroupDialog: false,
      groupTableData: [],
      editColumnDialogNames: "编辑列名",
      editColumnDialog: false,
      editedColumnName: "", // 要被编辑的列名
      oldColumnName: "", // 原来的列名
      addColumnDialogNames: "增加列",
      addColumnDialog: false,
      addedColumnName: "", // 增加的列名
      editItemDialogName: "编辑Item",
      editItemDialog: false,
      editItems: [], // 编辑框的Item
      rowId: 0, // 编辑行在数据库中id
      size: "medium", // 尺寸
      width: "800px",
      chartContainerWidth: "900px",
      widthChart: "1000px",
      maxHeight: 1000,
      small: false,
      transferValue: [],
      transferData: [],
      deleteHistoryDialogName: "历史删除",
      deleteHistoryDialog: false,
      titles: ["已有Item", "已选Item"],
      loadingTransfer: false,
      selectedRows: [], // 选中的行
      deletedRows: [], // 要被删除历史的items
      dst: [
        new Date(new Date().setTime(new Date() - 3600 * 1000 * 24)),
        new Date(),
      ], // 时间选择器的时间
      infoConfirmnDialog: false,
      itemInfos: [],
      timeStamp: "",
      deleteHistoryLoading: false
    };
  },
  mounted() {
    document.querySelector(".el-main").style.backgroundColor = " #ffffff";
  },
  created() {
    // 页面实例挂载完之后执行
    if (document.body.clientWidth < 768) {
      this.size = "mini";
      this.width = document.body.clientWidth + "px";
      this.chartContainerWidth = this.width;
      this.widthChart = document.body.clientWidth + "px";
      this.small = true;
      this.maxHeight = 600;
      this.rowCount = 9;
    }
    this.initial(true);
    this.actionUrl = "http://" + getCookie("ip") + "/page/uploadFile";
  },
  methods: {
    initial(flag = false) {
      getGroups().then(({ data }) => {
        this.groups = data.groupNames;
        if (flag) {
          this.selectedGroups = this.groups[0];
        }
        this.handleDeleteGroupButtonState =
          this.selectedGroups === "calc" ? true : false;
        this.render();
      });
    },
    // 点击加组按钮
    openGroupDialog() {
      this.groupForm.groupName = "";
      this.groupForm.columnName = "";
      this.groupDialog = true;
    },
    // 加组
    handleAddGroups() {
      if (this.groupForm.groupName.length != 0) {
        const g = this.groupForm.groupName.split(","); // 获取所有的组名
        const c = this.groupForm.columnName.split("|"); // 对应的列名
        if (g.length != c.length) {
          this.$message.error("列名个数和组名个数必须一致");
        } else {
          const d = [];
          for (let i = 0; i < g.length; i++) {
            if (c[i].length == 0) {
              d.push({ groupName: g[i] });
            } else {
              d.push({ groupName: g[i], columnNames: c[i].split(",") });
            }
          }
          addGroups(JSON.stringify(d))
            .then((r) => {
              this.$message.success("添加成功!");
              this.groupDialog = false;
              getGroups().then(({ data }) => {
                this.groups = data.groupNames;
                this.selectedGroups = this.groups[0];
              });
            })
            .catch(
              ({
                response: {
                  data: { message: message },
                },
              }) => {
                this.$notify.error({
                  title: "添加失败",
                  message,
                });
              }
            );
        }
      } else {
        this.$message.warning("请输入有效的组名");
      }
    },
    // 组切换回调函数
    getSelectedGroupInfos() {
      this.render(); // 重新渲染
      this.handleDeleteGroupButtonState =
        this.selectedGroups === "calc" ? true : false;
    },
    // 加单个点的弹窗
    openItemDialog() {
      this.itemDialog = true;
    },
    // 加单个点
    handleAddItem() {
      let values = [];
      let d = {};
      const f = this.$refs.itemForm;
      for (let i = 0; i < this.groupColumns.length; i++) {
        d[this.groupColumns[i]["label"]] = f[i].value;
      }
      values.push(d);
      addItem(
        JSON.stringify({
          groupName: this.selectedGroups,
          values,
        })
      )
        .then(() => {
          this.itemDialog = false;
          this.$message.success("添加成功!");
          this.render();
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.$notify.error({
              title: "添加失败",
              message,
            });
          }
        );
    },
    // 删除组
    handleGroup() {
      this.$confirm("删除操作不可逆,是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          deleteGroup(
            JSON.stringify({
              groupNames: [this.selectedGroups],
            })
          ).then(() => {
            getGroups()
              .then(({ data }) => {
                this.groups = data.groupNames;
                this.selectedGroups = this.groups[0];
                this.render();
                this.$message.success("删除成功!");
              })
              .catch(
                ({
                  response: {
                    data: { message: message },
                  },
                }) => {
                  this.$notify.error({
                    title: "删除失败",
                    message,
                  });
                }
              );
          });
        })
        .catch(() => {});
    },
    // 显示对应组的item数据
    showItems(groupName) {
      this.tableLoading = true;
      getItems(
        JSON.stringify({
          groupName: this.selectedGroups,
          column: "*",
          condition: this.searchCondition,
          startRow: this.startRow,
          rowCount: this.rowCount,
        })
      )
        .then(({ data: { itemCount, itemValues } }) => {
          this.itemCount = itemCount;
          this.itemData = [];
          let itemNames = [];
          if (itemValues != null) {
            for (let i = 0; i < itemValues.length; i++) {
              itemNames.push(itemValues[i].itemName);
            }
          }
          getRealTimeData(
            JSON.stringify({
              groupName,
              itemNames,
            })
          )
            .then((response) => {
              const timeData = response.data;
              for (let i = 0; i < itemNames.length; i++) {
                itemValues[i]["realTimeData"] =
                  timeData[itemNames[i]] === null ? "" : timeData[itemNames[i]];
                itemValues[i]["index"] = (this.startRow + i).toString();
              }
              this.itemData = itemValues;
              this.tableLoading = false;
            })
            .catch(
              ({
                response: {
                  data: { message: message },
                },
              }) => {
                this.tableLoading = false;
                this.$notify.error({
                  title: "获取实时值失败",
                  message,
                });
              }
            );
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.tableLoading = false;
            this.$notify.error({
              title: "获取Item失败",
              message,
            });
          }
        );
    },
    // 渲染表头和加点弹窗
    render() {
      // 根据组去获取对应的列名的信息
      getGroupColumns(
        JSON.stringify({
          groupNames: [this.selectedGroups],
        })
      )
        .then(({ data }) => {
          this.showItems(this.selectedGroups);
          this.rowKey = "rowKey";
          const columns = data[this.selectedGroups].itemColumnNames;
          let c = [];
          let c1 = [{ prop: "index", label: "index" }]; // index列
          this.tableColumns = [];
          const width =
            (1600 - parseInt(this.opWidth) - parseInt(this.indexWidth)) /
            (columns.length + 1);
          for (let i = 0; i < columns.length; i++) {
            c.push({ index: i + 1, label: columns[i], model: "" });
            c1.push({ prop: columns[i], label: columns[i], width: width });
          }
          c1.push({
            prop: "realTimeData",
            label: "realTimeData",
            width: width,
          }); // 实时值
          c1.push({ prop: "op", label: "op" }); // 操作列
          this.groupColumns = c; // 更新groupColumn弹窗中form
          this.tableColumns = c1; // 更新表头
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.tableColumns = [];
            this.itemData = [];
            this.$notify.error({
              title: "初始化失败",
              message,
            });
          }
        );
    },
    // 批量加点按钮
    openItemsDialog() {
      this.itemsDialog = true;
      this.fileList = [];
    },
    // 上传之前
    beforeUpload(file) {
      this.fileContent = file; // 获取文件
      if (this.isExcel(this.fileContent.name)) {
        return true;
      } else {
        this.$message.error("非法的文件类型或者文件列表已满!");
        return false; // 返回false取消上传
      }
    },
    // 上传
    uploadFile() {
      const data = new FormData();
      const fileUps = this.fileContent;
      data.append("file", fileUps);
      axios({
        headers: {
          "Content-Type": "multipart/form-data",
        },
        url: this.actionUrl,
        data: data,
        method: "post",
      })
        .then(() => {
          this.$message.success("上传成功");
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.$notify.error({
              title: "上传失败",
              message,
            });
          }
        );
    },
    // 检查是否是excel
    isExcel(name) {
      return /\.(xlsx|xls|csv)$/.test(name);
    },
    // 点击确定开始加点
    handleAddItems() {
      this.addItemLoading = true;
      addItemsByExcel(
        JSON.stringify({
          fileName: this.fileContent.name,
          groupName: this.selectedGroups,
        })
      )
        .then(() => {
          this.addItemLoading = false;
          this.itemsDialog = false;
          this.render();
          this.$message.success("加点成功");
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.addItemLoading = false;
            this.$notify.error({
              title: "加点失败",
              message,
            });
          }
        );
    },
    // 页码数变化
    handleSizeChange(c) {
      this.rowCount = c;
      this.startRow = (this.currentItemPage - 1) * this.rowCount; // start index
      this.showItems(this.selectedGroups);
    },
    // 页码变化
    handlePageChange() {
      this.startRow = (this.currentItemPage - 1) * this.rowCount;
      this.showItems(this.selectedGroups);
    },
    // 表格搜索
    handleSearch() {
      this.searchCondition = `itemName like '%${this.searchKeyWord}%'`;
      this.showItems(this.selectedGroups);
    },
    // 点表下载
    handleItemsDownload() {
      this.$confirm(`下载当前${this.itemCount}条数据?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.tableLoadingText = "正在下载中...";
          this.tableLoading = true;
          getGroupColumns(
            JSON.stringify({
              groupNames: [this.selectedGroups],
            })
          ).then(({ data }) => {
            const headers = data[this.selectedGroups].itemColumnNames;
            getItems(
              JSON.stringify({
                groupName: this.selectedGroups,
                column: "*",
                condition: this.searchCondition,
                startRow: -1,
              })
            )
              .then(({ data: { itemValues } }) => {
                const workBook = new excelJs.Workbook(); // 创建工作簿
                const workSheet = workBook.addWorksheet("Sheet1"); // 添加工作表
                workSheet.addRow(headers); // 添加excel的表头
                if (itemValues != null) {
                  itemValues.forEach((item) => {
                    let row = [];
                    headers.forEach((key) => {
                      row.push(item[key]);
                    });
                    workSheet.addRow(row); // 将数据写入工作表
                  });
                }
                workBook.xlsx.writeBuffer().then((buf) => {
                  saveAs(
                    new Blob([buf]),
                    this.selectedGroups + "_" + new Date().getTime() + ".xlsx"
                  ); // 将数据写入excel
                }); // 将数据写入字节流
                this.$message.success("下载完成");
                this.tableLoadingText = "表格数据加载中...";
                this.tableLoading = false;
              })
              .catch(
                ({
                  response: {
                    data: { message: message },
                  },
                }) => {
                  this.$notify.error({
                    title: "数据下载失败",
                    message,
                  });
                }
              );
          });
        })
        .catch(() => {});
    },
    // 查看历史曲线
    handleHisroty({ itemName }) {
      this.historyDialog = true;
      this.selectedItem = itemName;
    },
    // 显示历史曲线
    showHistory() {
      this.chartLoading = true;
      const s = new Date(this.parseTime(this.st[0])).getTime() / 1000;
      const e = new Date(this.parseTime(this.st[1])).getTime() / 1000;
      getHistoryData(
        JSON.stringify({
          groupName: this.selectedGroups,
          itemNames: [this.selectedItem],
          startTimes: [s],
          endTimes: [e],
          intervals: [parseInt(this.interval)],
        })
      )
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.$notify.error({
              title: "获取历史值失败",
              message,
            });
          }
        )
        .then(({ data }) => {
          this.chartItemValues = data[this.selectedItem];
          if (
            this.myChart != null &&
            this.myChart != "" &&
            this.myChart != undefined
          ) {
            this.myChart.dispose(); //解决echarts dom已经加载的报错
          }
          this.myChart = this.$echarts.init(document.getElementById("main"));
          this.myChart.setOption({
            title: {
              text: this.selectedItem,
              padding: [10, 10, 10, 50],
            },
            tooltip: {
              trigger: "axis",
            },
            xAxis: {
              data: this.chartItemValues[0].map((item) => {
                return this.parseTime(new Date(parseInt(item) * 1000));
              }),
            },
            yAxis: {
              splitLine: {
                show: false,
              },
            },
            toolbox: {},
            width: "900px",
            dataZoom: [
              {
                startValue: this.parseTime(
                  new Date(parseInt(this.chartItemValues[0]) * 1000)
                ),
              },
              {
                type: "inside",
              },
            ],
            series: {
              name: this.selectedItem,
              type: "line",
              data: this.chartItemValues[1],
            },
          });
          this.chartLoading = false;
        });
    },
    handleOpen() {
      this.showHistory();
    },
    // 转换时间
    parseTime(t) {
      return `${t.getFullYear()}-${
        t.getMonth() + 1 < 10 ? "0" + (t.getMonth() + 1) : t.getMonth() + 1
      }-${t.getDate() < 10 ? "0" + t.getDate() : t.getDate()} ${
        t.getHours() < 10 ? "0" + t.getHours() : t.getHours()
      }:${t.getMinutes() < 10 ? "0" + t.getMinutes() : t.getMinutes()}:${
        t.getSeconds() < 10 ? "0" + t.getSeconds() : t.getSeconds()
      }`;
    },
    // 下载数据
    handleDownloadData() {
      this.chartLoading = true;
      const workBook = new excelJs.Workbook(); // 创建工作簿
      const workSheet = workBook.addWorksheet("Sheet1"); // 添加工作表
      const headers = ["Time", "Data"];
      workSheet.addRow(headers); // 添加excel的表头
      if (this.chartItemValues != null) {
        for (let i = 0; i < this.chartItemValues[0].length; i++) {
          let row = [];
          row.push(
            this.parseTime(
              new Date(parseInt(this.chartItemValues[0][i]) * 1000)
            )
          );
          row.push(this.chartItemValues[1][i]);
          workSheet.addRow(row); // 将数据写入工作表
        }
      }
      workBook.xlsx.writeBuffer().then((buf) => {
        saveAs(
          new Blob([buf]),
          this.selectedGroups +
            "_" +
            this.selectedItem +
            "_" +
            new Date().getTime() +
            ".xlsx"
        ); // 将数据写入excel
      }); // 将数据写入字节流
      this.$message.success("下载完成");
      this.chartLoading = false;
    },
    // 编辑此组
    handleEditGroup() {
      this.editGroupDialog = true;
      this.groupTableData = [];
      for (let item of this.groupColumns) {
        this.groupTableData.push({ columnName: item.label });
      }
    },
    // 文件列表已满
    showExceed() {
      this.$message.error("文件列表已满");
    },
    // 编辑column
    editColumns({ columnName }) {
      this.editColumnDialog = true;
      this.oldColumnName = columnName;
      this.editedColumnName = columnName;
    },
    // 确定更新列名
    editColumnHandler() {
      if (this.oldColumnName != this.editedColumnName) {
        this.$confirm("确定更新列名?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        })
          .then(() => {
            updateColumnNames(
              JSON.stringify({
                groupName: this.selectedGroups,
                oldColumnNames: [this.oldColumnName],
                newColumnNames: [this.editedColumnName],
              })
            )
              .then(({ data }) => {
                this.groupTableData.map((item) => {
                  item.columnName === this.oldColumnName
                    ? (item.columnName = this.editedColumnName)
                    : "";
                });
                this.$message.success("更新成功!");
                this.editColumnDialog = false;
              })
              .catch(
                ({
                  response: {
                    data: { message: message },
                  },
                }) => {
                  this.$notify.error({
                    title: "更新失败",
                    message,
                  });
                  this.editColumnDialog = false;
                }
              );
          })
          .catch(() => {});
      } else {
        this.editColumnDialog = false;
      }
    },
    // 点击增加列
    addColumns() {
      this.addedColumnName = "";
      this.addColumnDialog = true;
    },
    // 删除列
    deleteColumns({ columnName }) {
      this.$confirm("删除操作不可逆,是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          deleteColumns(
            JSON.stringify({
              groupName: this.selectedGroups,
              columnNames: [columnName],
            })
          )
            .then(({ data }) => {
              this.groupTableData = this.groupTableData.filter((item) => {
                return item.columnName != columnName;
              });
              this.$message.success("删除成功!");
            })
            .catch(
              ({
                response: {
                  data: { message: message },
                },
              }) => {
                this.$notify.error({
                  title: "删除列失败",
                  message,
                });
              }
            );
        })
        .catch(() => {});
    },
    // 关闭增加此组的弹窗
    editGroupDialogCloseHandler() {
      this.initial();
      this.editGroupDialog = false;
    },
    // 确定增加列
    addColumnHandler() {
      if (this.addedColumnName.length === 0) {
        this.addColumnDialog = false;
      } else {
        addColumns(
          JSON.stringify({
            groupName: this.selectedGroups,
            columnNames: this.addedColumnName.split(",").map((item) => {
              return item.trim();
            }),
          })
        )
          .then(() => {
            for (let item of this.addedColumnName.split(",")) {
              this.groupTableData.push({ columnName: item.trim() });
            }
            this.addColumnDialog = false;
          })
          .catch(
            ({
              response: {
                data: { message: message },
              },
            }) => {
              this.$notify.error({
                title: "增加列失败",
                message,
              });
              this.addColumnDialog = false;
            }
          );
      }
    },
    // 编辑item
    editItem(row) {
      this.rowId = row.id;
      this.editItems = [];
      this.editItemDialog = true;
      for (let k in row) {
        if (
          k != "id" &&
          k != "index" &&
          k != "realTimeData" &&
          k != "itemName"
        ) {
          this.editItems.push({ label: k, value: row[k] });
        }
      }
    },
    // 确定编辑item按钮
    editItemsHandler() {
      let clause = [];
      let condition = "id=" + this.rowId;
      for (let i = 0; i < this.editItems.length; i++) {
        clause.push(`${this.editItems[i].label}='${this.editItems[i].value}'`);
      }
      updateItems(
        JSON.stringify({
          groupName: this.selectedGroups,
          clause: clause.join(","),
          condition,
        })
      )
        .then(() => {
          this.$message.success("更新成功!");
          this.render();
          this.editItemDialog = false;
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.$notify.error({
              title: "更新失败",
              message,
            });
          }
        );
    },
    // 删除item
    deleteItem({ itemName }) {
      this.$confirm("删除操作不可逆,是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          deleteItems(
            JSON.stringify({
              groupName: this.selectedGroups,
              condition: "itemName='" + itemName + "'",
            })
          )
            .then(() => {
              this.$message.success("删除成功!");
              this.initial();
            })
            .catch(
              ({
                response: {
                  data: { message: message },
                },
              }) => {
                this.$notify.error({
                  title: "删除失败",
                  message,
                });
              }
            );
        })
        .catch(() => {});
    },
    // 删除历史
    handleDeleteHistory() {
      if (this.selectedRows.length === 0) {
        this.$message.error("请选择要删除历史的items");
      } else {
        this.$confirm("删除历史之前请确认已关闭其他写数程序?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        })
          .then(() => {
            this.deleteHistoryDialog = true;
          })
          .catch();
      }
    },
    // 设置穿梭框的宽度,并且加载穿梭框中的数据
    changeTransferWidth() {
      document.querySelectorAll(".el-transfer-panel").forEach((item) => {
        item.style.width = "350px";
        item.style.textAlign = "left";
      });
      this.transferValue = [];
      this.transferData = [];
      for (let i = 0; i < this.selectedRows.length; i++) {
        this.transferData.push({
          key: i,
          label: this.selectedRows[i]["itemName"],
        });
      }
      this.deletedRows = this.selectedRows;
      this.titles = [
        `${this.selectedGroups}中已选择的Items`,
        `${this.selectedGroups}中已剔除的Items`,
      ];
    },
    // 表格选中的数据
    handleSelection(s) {
      this.selectedRows = s;
    },
    // 点击删除历史
    deleteHistory() {
      this.timeStamp = `${this.parseTime(this.dst[0])} - ${this.parseTime(
        this.dst[1]
      )}`;
      this.itemInfos = [];
      if (this.deletedRows.length > 5) {
        for (let i = 0; i < 5; i++) {
          this.itemInfos.push(this.deletedRows[i]);
        }
      } else {
        this.itemInfos = this.deletedRows;
      }
      this.infoConfirmnDialog = true;
    },
    //
    updateDeletedRows(indexes) {
      this.deletedRows = [];
      for (let i = 0; i < this.selectedRows.length; i++) {
        if (indexes.indexOf(i) === -1) {
          this.deletedRows.push({ itemName: this.selectedRows[i]["itemName"] }); // update deleted rows
        }
      }
    },
    loadItems() {
      const t = _.difference(this.deletedRows, this.itemInfos, "itemName");
      if (t.length <= 2) {
        this.itemInfos = this.itemInfos.concat(t);
      } else {
        for (let i = 0; i < 2; i++) {
          this.itemInfos.push(t[i]);
        }
      }
    },
    confirmDeleteHistory() {
      // new Date(this.parseTime(this.st[0])).getTime() / 1000
      const startTime = `${new Date(this.parseTime(this.dst[0])).getTime() / 1000}`;
      const endTime = `${new Date(this.parseTime(this.dst[1])).getTime() / 1000}`;
      const itemNames = this.deletedRows.map((item) => {
        return item.itemName;
      });
      const timeStamps = this.deletedRows.map((item) => {
        return { startTime, endTime };
      });
      this.deleteHistoryLoading = true
      deleteHistory(
        JSON.stringify({
          itemNames,
          timeStamps,
        })
      )
        .then(() => {
          this.deleteHistoryLoading = false
          this.infoConfirmnDialog = false
          this.deleteHistoryDialog = false
          this.$message.success("历史删除成功");
        })
        .catch(
          ({
            response: {
              data: { message: message },
            },
          }) => {
            this.deleteHistoryLoading = false
            this.$notify.error({
              title: "删除失败",
              message,
            });
          }
        );
    },
  },
};
</script>
<style scoped>
.el-select {
  width: 100%;
}
.el-autocomplete {
  width: 100%;
}
/* 设置分页的样式 */
.el-pagination {
  float: left;
}
.el-upload-dragger {
  width: 600px;
}
.el-divider--horizontal {
  margin: 10px 0;
  width: 100%;
}
.infinite-list {
  height: 300px;
  padding: 0;
  margin: 0;
  list-style: none;
}

.demo-block {
  border: 1px solid #ebebeb;
  border-radius: 3px;
  transition: 0.2s;
}

.demo-block.hover {
  box-shadow: 0 0 8px 0 rgba(232, 237, 250, 0.6),
    0 2px 4px 0 rgba(232, 237, 250, 0.5);
}

.demo-block code {
  font-family: Menlo, Monaco, Consolas, Courier, monospace;
}

.demo-block .demo-button {
  float: right;
}

.demo-block .source {
  padding: 24px;
}

.demo-block .meta {
  background-color: #fafafa;
  border-top: 1px solid #eaeefb;
  overflow: hidden;
  height: 0;
  transition: height 0.2s;
}

.demo-block .description {
  padding: 20px;
  box-sizing: border-box;
  border: 1px solid #ebebeb;
  border-radius: 3px;
  font-size: 14px;
  line-height: 22px;
  color: #666;
  word-break: break-word;
  margin: 10px;
  background-color: #fff;
}

.demo-block .description p {
  margin: 0;
  line-height: 26px;
}

.demo-block .description code {
  color: #5e6d82;
  background-color: #e6effb;
  margin: 0 4px;
  display: inline-block;
  padding: 1px 5px;
  font-size: 12px;
  border-radius: 3px;
  height: 18px;
  line-height: 18px;
}

.demo-block .highlight pre {
  margin: 0;
}

.demo-block .highlight code.hljs {
  margin: 0;
  border: none;
  max-height: none;
  border-radius: 0;
}

.demo-block .highlight code.hljs:before {
  content: none;
}

.demo-block .demo-block-control {
  border-top: 1px solid #eaeefb;
  height: 44px;
  box-sizing: border-box;
  background-color: #fff;
  border-bottom-left-radius: 4px;
  border-bottom-right-radius: 4px;
  text-align: center;
  margin-top: -1px;
  color: #d3dce6;
  cursor: pointer;
  position: relative;
}

.demo-block .demo-block-control.is-fixed {
  position: fixed;
  bottom: 0;
  width: 868px;
}

.demo-block .demo-block-control i {
  font-size: 16px;
  line-height: 44px;
  transition: 0.3s;
}

.demo-block .demo-block-control i.hovering {
  transform: translateX(-40px);
}

.demo-block .demo-block-control > span {
  position: absolute;
  transform: translateX(-30px);
  font-size: 14px;
  line-height: 44px;
  transition: 0.3s;
  display: inline-block;
}

.demo-block .demo-block-control:hover {
  color: #409eff;
  background-color: #f9fafc;
}

.demo-block .demo-block-control .text-slide-enter,
.demo-block .demo-block-control .text-slide-leave-active {
  opacity: 0;
  transform: translateX(10px);
}

.demo-block .demo-block-control .control-button {
  line-height: 26px;
  position: absolute;
  top: 0;
  right: 0;
  font-size: 14px;
  padding-left: 5px;
  padding-right: 25px;
}

..infinite-list {
  height: 300px;
  padding: 0;
  margin: 0;
  list-style: none;
}

.infinite-list .infinite-list-item {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 50px;
  background: #e8f3fe;
  margin: 10px;
  color: #7dbcfc;
}

.infinite-list .infinite-list-item + .list-item {
  margin-top: 10px;
}

.infinite-list-wrapper {
  height: 300px;
  text-align: center;
}

.infinite-list-wrapper .list {
  padding: 0;
  margin: 0;
  list-style: none;
}

.infinite-list-wrapper .list-item {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 50px;
  background: #fff6f6;
  color: #ff8484;
}

.infinite-list-wrapper .list-item + .list-item {
  margin-top: 10px;
}
</style>