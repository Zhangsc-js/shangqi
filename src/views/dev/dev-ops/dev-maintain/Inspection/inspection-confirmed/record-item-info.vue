<template>
  <div>
    <el-form ref="itemInfoForm" :model="itemInfoForm" label-width="200px">
      <el-divider content-position="center">基础信息</el-divider>
      <el-row>
        <el-col :span="12">
          <el-form-item label="设备名称:" prop="devName">
            <span>{{ itemInfoForm.devName }}</span>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="部位名称:" prop="partsName">
            <span>{{ itemInfoForm.item.partsName }}</span>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="巡检项目:" prop="projectName">
            <span>{{ itemInfoForm.item.projectName }}</span>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="巡检方法:" prop="methodName">
            <span>{{ itemInfoForm.item.methodName }}</span>
          </el-form-item>
        </el-col>
      </el-row>
      <!-- <el-row>
        <el-col :span="12">
          <el-form-item label="开始时间:" prop="startTime" >
            <span>{{ itemInfoForm.startTime }}</span>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="结束时间:" prop="endTime" >
            <span>{{ itemInfoForm.endTime }}</span>
          </el-form-item>
        </el-col>
      </el-row>-->
      <el-row>
        <!-- <el-col :span="12">
          <el-form-item label="用时:" prop="tokeTime" >
            <span>{{ itemInfoForm.tokeTime }}</span>
          </el-form-item>
        </el-col>-->
        <el-col :span="12">
          <el-form-item label="状态:" prop="status">
            <span v-if="itemInfoForm.status == 1">{{ '正常'}}</span>
            <span v-else-if="itemInfoForm.status == 8">{{ '已上报'}}</span>
            <span v-else-if="itemInfoForm.status == 9">{{ '异常'}}</span>
          </el-form-item>
        </el-col>
      </el-row>
      <el-divider content-position="center">异常信息</el-divider>
      <!-- <el-row>
        <el-col :span="24">
          <el-form-item label="异常原因:" prop="exceptionReason">
            <span>{{ itemInfoForm.exceptionReason }}</span>
          </el-form-item>
        </el-col>
      </el-row> -->
      <!-- <el-row>
        <el-col :span="24">
          <el-form-item label="异常处理方法:" prop="exceptionHandleMethod" >
            <span>{{ itemInfoForm.exceptionHandleMethod }}</span>
          </el-form-item>
        </el-col>
      </el-row>-->
      <el-row>
        <el-col :span="24">
          <el-form-item label="处理情况:" prop="exceptionHandleResult">
            <span>{{ itemInfoForm.exceptionHandleResult }}</span>
          </el-form-item>
        </el-col>
      </el-row>
      <el-divider content-position="center">实时数据</el-divider>
      <el-row>
        <el-col :span="12">
          <el-form-item label="现场情况:" prop="realtimeData">
            <span>{{ itemInfoForm.realtimeData }}</span>
          </el-form-item>
        </el-col>
        <!-- <el-col :span="12">
          <el-form-item label="照片" prop="photo">
            <div class="demo-image__preview">
              <el-image
                style="width: 300px; height: 300px"
                :src="itemInfoForm.photo"
                :preview-src-list="srcList"
              ></el-image>
            </div>
          </el-form-item>
        </el-col> -->
      </el-row>
      <el-row>
          <el-col :span="8" v-for="item in srcList" :key="item">
            <el-form-item label="照片" prop="item" >
              <div class="demo-image__preview">
                <el-image
                  style="width: 300px; height: 300px"
                  :src="item">
                </el-image>
              </div>
            </el-form-item>
          </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<script>
import {
  getFileList } from '@/api/device'
export default {
  name: "ItemInfo",
  data() {
    return {
      itemInfoForm: {},
      disabled: true,
      srcList: []
    };
  },
  props: {
    row: {
      type: Object,
      required: true
    }
  },
  watch: {
    row() {
      this.getData();
    }
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      this.itemInfoForm = this.row;
      this.srcList = []
      if (this.itemInfoForm.photo) {
        const params = { ids: this.itemInfoForm.photo };
        getFileList(params)
          .then(response => {
            const result = response.data;
            if (result.success) {
              const imgServer = process.env.VUE_APP_DEV_IMAGE_URL;
              for (const e of result.data) {
                this.srcList.push(imgServer + e.uploadName);
              }
              console.log(this.srcList)
            } else {
              this.$message.error(result.message);
            }
          })
          .catch(e => {
            this.$message.error(e.message);
          });
      }
    }
  }
};
</script>
