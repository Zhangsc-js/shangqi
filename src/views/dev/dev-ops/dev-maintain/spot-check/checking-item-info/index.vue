<template>
  <div style="height: 100%">
    <div>
      <el-form inline ref="queryForm" :model="queryForm" style="margin-left:20px">
        <el-form-item label="设备名称" prop="devName">
          <el-input v-model="queryForm.devName" placeholder="请输入设备名称" />
        </el-form-item>
        <el-form-item>
          <el-button icon="el-icon-search" type="primary" @click="getData(1)">查询</el-button>
          <el-button
            href="javascript:void(0)"
            @click="clearSearchBox"
            icon="el-icon-refresh-left"
            type="primary"
          >重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div>
      <el-row style="margin:10px 0 0 20px">
        <el-button
          v-has="'DEV-OPS-CHECK-ITEM-INFO-ADD'"
          type="primary"
          @click="addRow"
          icon="el-icon-plus"
        >新增</el-button>
        <el-button
          v-has="'DEV-OPS-CHECK-ITEM-INFO-DNOWLOAD'"
          type="primary"
          @click="downloadTemlate"
          icon="el-icon-download"
        >模版下载</el-button>
        <el-upload
          class="upload"
          :action="fileUploadUrl"
          :show-file-list="false"
          :headers="token"
          :on-success="uploadSuccess"
          :on-error="uploadError"
          :on-progress="progress"
        >
          <el-button
            v-has="'DEV-OPS-CHECK-ITEM-INFO-UPLOAD'"
            type="primary"
            icon="el-icon-upload2"
          >批量导入</el-button>
        </el-upload>
        <el-button
          v-has="'DEV-OPS-CHECK-ITEM-INFO-BATDELETE'"
          type="danger"
          :disabled="batchBtnVisable"
          @click="batchDel"
        >批量删除</el-button>
      </el-row>
    </div>
    <el-table
      stripe
      border
      :data="tableData"
      @selection-change="handleSelectionChange"
      style="width: 100%"
    >
      <el-table-column fixed type="selection" width="55"></el-table-column>
      <el-table-column prop="devName" label="设备名称" align="center">
        <template v-slot="scope">
          <el-input
            v-model="scope.row.devName"
            v-if="scope.row.seen"
            @click.native="devNameRowCliek(scope.row)"
          ></el-input>
          <span v-else>{{ scope.row.devName }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="itemInfoNo" label="点检项信息编号" align="center">
        <template v-slot="scope">
          <el-input v-model="scope.row.itemInfoNo" v-if="scope.row.seen && scope.row.canEdit"></el-input>
          <span v-else>{{ scope.row.itemInfoNo }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="partsName" label="点检部位" align="center">
        <template v-slot="scope">
          <el-input v-model="scope.row.partsName" v-if="scope.row.seen"></el-input>
          <span v-else>{{ scope.row.partsName }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="projectName" label="点检内容" align="center">
        <template v-slot="scope">
          <el-input v-model="scope.row.projectName" v-if="scope.row.seen"></el-input>
          <span v-else>{{ scope.row.projectName }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="methodName" label="点检方法" align="center">
        <template v-slot="scope">
          <el-input v-model="scope.row.methodName" v-if="scope.row.seen"></el-input>
          <span v-else>{{ scope.row.methodName }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="criteriaName" label="点检标准" align="center">
        <template v-slot="scope">
          <el-input v-model="scope.row.criteriaName" v-if="scope.row.seen"></el-input>
          <span v-else>{{ scope.row.criteriaName }}</span>
        </template>
      </el-table-column>
      <el-table-column
        v-if="hasBtn([
          'DEV-OPS-CHECK-ITEM-INFO-UPDATE',
          'DEV-OPS-CHECK-ITEM-INFO-DELETE'
        ])"
        align="center"
        label="操作"
        fixed="right"
        width="120px"
      >
        <template v-slot="scope">
          <el-button
            v-has="'DEV-OPS-CHECK-ITEM-INFO-UPDATE'"
            type="text"
            v-if="!scope.row.visibleCancel"
            size="small"
            @click.stop="edit(scope.row)"
          >更新</el-button>
          <el-button
            type="text"
            v-if="scope.row.visibleCancel"
            size="small"
            @click.stop="save(scope.row)"
          >保存</el-button>
          <el-button
            type="text"
            v-if="scope.row.visibleCancel"
            size="small"
            @click.stop="cancel(scope.row)"
          >取消</el-button>
          <el-button
            v-has="'DEV-OPS-CHECK-ITEM-INFO-DELETE'"
            type="text"
            v-if="scope.row.id && !scope.row.visibleCancel"
            size="small"
            @click.stop="deleteRow(scope.row.id)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <Pagination
      :total="total"
      :page.sync="page.pageNum"
      :limit.sync="page.pageSize"
      @pagination="getData"
    />
    <el-dialog
      title="设备树选择"
      :visible.sync="dialogVisible"
      :modal-append-to-body="false"
      width="65%"
    >
      <select-tree
        :requestUrl="requestUrl"
        :queryParams="queryParams"
        @saveSelectNode="handleSaveSelectNode"
        :multipleSelection="multipleSelection"
        style="height: 100%"
      ></select-tree>
    </el-dialog>
  </div>
</template>

<script>
import SelectTree from "@/components/SelectTree";
import Pagination from "@/components/Pagination";

import {
  getCheckingItemInfos,
  getDevTree,
  updateCheckingItemInfo,
  addCheckingItemInfo,
  deleteCheckingItemInfo,
  batchDelCheckingItemInfos,
  IMPORT_CHECKING_ITEMS,
  downloadCheckingItemTemp
} from "@/api/device";
import { isEmptyArray, hasBtn } from "@/utils/index";
import { saveAs } from "file-saver";
import { getToken } from "@/utils/auth";

export default {
  name: "CheckingItemInfo",
  components: {
    SelectTree,
    Pagination
  },
  data() {
    return {
      loading: null,
      page: {
        pageNum: 1,
        pageSize: 10
      },
      total: 0,
      tableData: [],
      opts: [],
      selectIds: [],
      dialogVisible: false,
      batchBtnVisable: true,
      requestUrl: getDevTree,
      multipleSelection: null, // 树是否支持多选
      selectTreeIds: [], // 弹出设备树多选ID
      selectRow: {},
      queryForm: {
        devName: ""
      },
      rules: {
        tempName: [
          { required: true, message: "请选输入设备名称", trigger: "change" }
        ]
      },
      fileUploadUrl: IMPORT_CHECKING_ITEMS,
      queryParams: {
        isInspection: 1
      },
      token: {
        Authorization: `Bearer ${getToken()}`
      }
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    hasBtn,
    getData(pageNum) {
      if (pageNum === 1) {
        this.page.pageNum = pageNum;
      }
      const params = {
        ...this.page,
        ...this.queryForm
      };
      getCheckingItemInfos(params)
        .then(response => {
          const result = response.data;
          if (result.success) {
            this.tableData = result.data.rows;
            this.total = result.data.total;
            this.setEditProperty();
          } else {
            this.$message.error(result.message);
          }
        })
        .catch(e => {
          this.$message.error(e.message);
        });
    },
    addRow() {
      this.dialogVisible = true;
      this.multipleSelection = true;
    },
    /**
     * 下载模板
     */
    downloadTemlate() {
      downloadCheckingItemTemp()
        .then(response => {
          const result = response.data;
          if (result) {
            let data = new File([result], { type: "application/vnd.ms-excel" });
            saveAs(data, "点检项目信息导入模版.xls");
          } else {
            this.$message.error(result.message);
          }
        })
        .catch(e => {
          this.$message.error(e.message);
        });
    },
    /**
     * 批量删除
     */
    batchDel() {
      if (isEmptyArray(this.selectIds)) {
        return;
      }
      this.$confirm("此操作将永久删除该记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          batchDelCheckingItemInfos(this.selectIds).then(response => {
            const result = response.data;
            if (result.success) {
              this.$message.success("批量删除成功!");
              this.getData(1);
            } else {
              this.$message.error(result.message);
            }
          });
        })
        .catch(() => {
          this.$message.info("已取消删除");
        });
    },
    edit(row) {
      row.visibleCancel = true;
      row.seen = true;
    },
    save(row) {
      if (this.formValidate(row)) {
        return;
      }
      if (row.id) {
        updateCheckingItemInfo(row)
          .then(response => {
            const result = response.data;
            if (result.success) {
              this.$message.success("更新成功");
              this.getData();
            } else {
              this.$message.error(result.message);
            }
          })
          .catch(e => {
            this.$message.error(e.message);
          });
      } else {
        addCheckingItemInfo(row)
          .then(response => {
            const result = response.data;
            if (result.success) {
              this.$message.success("新增成功");
              this.getData();
            } else {
              this.$message.error(result.message);
            }
          })
          .catch(e => {
            this.$message.error(e.message);
          });
      }
    },
    cancel(row) {
      row.visibleCancel = false;
      row.seen = false;
      this.getData();
    },
    deleteRow(id) {
      this.$confirm("此操作将永久删除该记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          deleteCheckingItemInfo(id).then(response => {
            const result = response.data;
            if (result.success) {
              this.$message.success("删除成功");
              this.getData(1);
            } else {
              this.$message.error(result.message);
            }
          });
        })
        .catch(() => {
          this.$message.info("已取消删除");
        });
    },
    /**
     * 弹出框保存事件
     */
    handleSaveSelectNode(data) {
      this.dialogVisible = false;
      // 当新增可以多选并且会在表格中新增行
      if (this.multipleSelection) {
        for (let e of data) {
          this.tableData.unshift({
            devName: e.label,
            devNo: e.code,
            seen: true,
            visibleCancel: true,
            canEdit: true
          });
        }
      } else {
        // 当非多选时为更新，利用浅拷贝更改表格中的值
        this.selectRow.devName = data[0].label;
        this.selectRow.devNo = data[0].code;
      }
    },
    progress() {
      this.loading = this.$loading({
        lock: true,
        text: "上传中",
        background: "rgba(0, 0, 0, 0.7)"
      });
    },
    /**
     * 选择行变化
     */
    handleSelectionChange(data) {
      this.selectIds = [];
      for (let e of data) {
        this.selectIds.push(e.id);
      }
      this.batchBtnVisable = isEmptyArray(this.selectIds);
    },
    /**
     * 设置可编辑属性
     */
    setEditProperty() {
      if (!isEmptyArray(this.tableData)) {
        this.tableData.map(v => {
          this.$set(v, "visibleCancel", false);
          this.$set(v, "seen", false);
          return v;
        });
      }
    },
    devNameRowCliek(row) {
      this.dialogVisible = true;
      this.multipleSelection = false;
      this.selectRow = row;
    },
    uploadSuccess(response) {
      this.loading.close();
      if (response.success) {
        this.$message.success("导入成功");
      } else {
        this.$message({
            message: response.message,
            type: 'error',
            duration: 8 * 1000
        })
      }
      this.getData();
    },
    uploadError(err) {
      this.loading.close();
      this.$message.error("上传失败" + err.message);
    },
    clearSearchBox() {
      this.$refs["queryForm"].resetFields();
      this.getData();
    },
    formValidate(row) {
      if (!row.devName) {
        this.$message.error("请选择设备名称");
        return true;
      }
      if (!row.itemInfoNo) {
        this.$message.error("请填写点检项编码");
        return true;
      }
      if (!row.partsName) {
        this.$message.error("请填写点检部位");
        return true;
      }
      if (!row.projectName) {
        this.$message.error("请填写点检内容");
        return true;
      }
      if (!row.methodName) {
        this.$message.error("请填写点检方法");
        return true;
      }
      if (!row.criteriaName) {
        this.$message.error("请填写点检标准");
        return true;
      }
      return false;
    }
  }
};
</script>
<style lang="scss" scoped >
.upload {
  display: inline-block;
  margin: 0 7px;
}
</style>
