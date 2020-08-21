<template>
  <div class="home" style="margin-top: 20px">
    <el-select v-model="value" placeholder="请选择要下载的标准">
      <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"></el-option>
    </el-select>
    <el-input
      v-model="stdName"
      placeholder="请输入要搜索的标准"
      style="width:202px;margin:0px 5px"
      v-on:keyup.enter.native="search"
    ></el-input>
    <el-button type="primary" icon="el-icon-search" @click="search">搜索</el-button>
    <el-table
      ref="multipleTable"
      :data="tableData"
      tooltip-effect="dark"
      style="width: 100%"
      v-loading="loading"
      element-loading-text="拼命加载中"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.1)"
    >
      <el-table-column prop="standard_no" label="标准号" width="500"></el-table-column>
      <el-table-column prop="name" label="标准" width="500"></el-table-column>
      <el-table-column prop="status" label="状态" show-overflow-tooltip></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" @click="download(scope.row.key, scope.row.name)">下载</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import axios from "axios";
import { stat } from "fs";
import FileSaver from "file-saver";

@Component
export default class Home extends Vue {
  loading = false;
  aa = 1;
  options = [
    {
      value: "gb",
      label: "国标",
    },
    {
      value: "db",
      label: "地标",
    },
    {
      value: "hb",
      label: "行标",
    },
  ];
  value = "";
  stdName = "";
  data = [];
  instance = axios.create({
    baseURL: "https://localhost:23439",
    timeout: 50000,
  });
  tableData = [];

  search() {
    if (this.value && this.stdName) {
      this.getStdInfo(this.value, { q: this.stdName });
    } else {
      this.$message({
        message: "请选择标准类型和标准名",
        showClose: true,
        type: "error",
      });
    }
  }

  getStdInfo(t: string, params: any) {
    this.loading = true;
    axios
      .get(`http://localhost:23439/api/search/${t}`, {
        params: params,
      })
      .then((res) => res.data)
      .then((res) => {
        this.tableData = res.records;
        this.loading = false;
        console.log(res);
      });
    return " ";
  }
  async _download(key: string, name: string) {
    const statusData = await axios.get(
      `http://localhost:23439/api/download_check/gb`,
      {
        params: {
          key: key,
        },
      }
    );
    if (statusData.data.status === 1) {
      this.$message("开始下载");
      const fileBlobRes = await axios.get(
        `http://localhost:23439/api/download/gb`,
        {
          params: {
            key: key,
          },
          responseType: "blob",
        }
      );
      const fileBlob = fileBlobRes.data;
      FileSaver.saveAs(new Blob([fileBlob]), name);
      return Promise.resolve();
    } else {
      return Promise.reject();
    }
  }

  download(key: string, name: string) {
    this._download(key, name)
      .then(() => this.$message.success("下载成功"))
      .catch(() => this.$message.error("该标准暂不支持下载"));
  }
}
</script>