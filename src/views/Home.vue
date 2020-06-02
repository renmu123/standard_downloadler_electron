<template>
  <div class="home">
    <el-select v-model="value" placeholder="请选择要下载的标准">
      <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"></el-option>
    </el-select>
    <el-row type="flex" justify="center" style="margin-top:10px">
      <el-col :span="4">
        <el-input v-model="stdName" placeholder="请输入要搜索的标准"></el-input>
      </el-col>
      <el-col :span="2">
        <el-button type="primary" icon="el-icon-search" @click="search">搜索</el-button>
      </el-col>
    </el-row>
    <el-table ref="multipleTable" :data="tableData" tooltip-effect="dark" style="width: 100%">
      <el-table-column prop="standard_no" label="标准号" width="500"></el-table-column>
      <el-table-column prop="name" label="标准" width="500"></el-table-column>
      <el-table-column prop="status" label="状态" show-overflow-tooltip></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" @click="download(scope.row.key)">下载</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import axios from "axios";

@Component
export default class Home extends Vue {
  aa = 1;
  options = [
    {
      value: "gb",
      label: "国标"
    },
    {
      value: "db",
      label: "地标"
    },
    {
      value: "hb",
      label: "行标"
    }
  ];
  value = "";
  stdName = "";
  data = [];
  instance = axios.create({
    baseURL: "https://localhost:5000",
    timeout: 50000
  });
  tableData = [];

  search() {
    this.getStdInfo(this.value, { q: this.stdName });
  }

  getStdInfo(t: string, params: any) {
    axios
      .get(`http://localhost:5000/api/search/${t}`, {
        params: params
      })
      .then(res => res.data)
      .then(res => {
        this.tableData = res.records;
        console.log(res);
      });
    return " ";
  }

  download(key: string) {
    axios
      .get(`http://localhost:5000/api/download/gb`, {
        params: {
          key: key
        },
        responseType: "blob"
      })
      .then(res => res.data)
      .then(res => {
        const url = window.URL.createObjectURL(new Blob([res]));
        const link = document.createElement("a");
        link.style.display = "none";
        link.href = url;
        link.setAttribute("download", "aa.pdf");

        document.body.appendChild(link);
        link.click();
        //释放URL对象所占资源
        window.URL.revokeObjectURL(url);
        //用完即删
        document.body.removeChild(link);

        // alert("下载成功");
      });
  }
}
</script>