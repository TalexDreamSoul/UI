
<template>
  <div v-loading="loading">
    <div style="display: block">
      <el-select
        v-model="selectedServiceUuid"
        filterable
        placeholder="选择远程地址"
        size="small"
        style="margin-right: 10px"
        @change="remoteSelectHandle"
      >
        <el-option
          v-for="item in serviceList"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        >
        </el-option>
      </el-select>
      <el-input
        v-model="instanceNameKeyword"
        placeholder="模拟名称"
        size="small"
        style="width: 180px; margin-right: 10px"
      ></el-input>
      <el-button size="small" @click="remoteSelectHandle">
        <i class="el-icon-search"></i> 搜索
      </el-button>
    </div>
    <div class="row-mt">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="maxPage"
        v-model:currentPage="page"
        :page-size="1"
        @current-change="handleCurrentChange"
        small
      ></el-pagination>
    </div>
    <div class="row-mt">
      <el-table :data="instances" stripe style="width: 100%" size="mini">
        <el-table-column prop="nickname" label="实例昵称" min-width="240"></el-table-column>
        <el-table-column label="操作" style="text-align: right" width="180">
          <template #default="scope">
            <el-button size="small" @click="callback(scope.row)"> 选择 </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
import { API_SERVICE_INSTANCES, API_SERVICE_LIST } from "../app/service/common";
import { statusCodeToText } from "../app/service/instance_tools";
import { request } from "../app/service/protocol";

export default {
  props: {
    callback: Function
  },

  data: function () {
    return {
      instances: [],
      serviceList: [],
      selectedServiceUuid: "",
      selectedServiceIp: "",
      instanceNameKeyword: "",

      page: 1,
      maxPage: 1,
      loading: true
    };
  },
  methods: {
    async renderServices() {
      try {
        const data = await request({
          method: "GET",
          url: API_SERVICE_LIST
        });
        for (const service of data) {
          const remarks = `${service.remarks}`;
          if (service.available) {
            const ip = `${service.ip}:${service.port}`;
            this.serviceList.push({
              value: `${service.uuid} ${ip}`,
              label: `${ip} ${remarks}`
            });
          } else {
            const ip = `${service.ip}:${service.port}`;
            this.serviceList.push({
              value: `${service.uuid} ${ip}`,
              label: `${ip} ${remarks} (离线)`
            });
          }
        }
      } catch (error) {
        this.$message({ type: "error", message: `错误:${error.message}` });
      } finally {
        this.loading = false;
      }
    },
    async remoteSelectHandle() {
      try {
        if (!this.selectedServiceUuid) throw new Error("还未选择远程守护进程");
        const hostIp = this.selectedServiceUuid.split(" ")[1];
        const serviceUuid = this.selectedServiceUuid.split(" ")[0];
        this.loading = true;
        const result = await request({
          method: "GET",
          url: API_SERVICE_INSTANCES,
          params: {
            remote_uuid: serviceUuid,
            page: this.page,
            page_size: 10,
            instance_name: this.instanceNameKeyword
          }
        });
        this.page = result.page;
        this.maxPage = result.maxPage;
        this.instances = [];
        const instances = result.data;
        instances.forEach((instance) => {
          this.instances.push({
            instanceUuid: instance.instanceUuid,
            serviceUuid: serviceUuid,
            nickname: instance.config.nickname,
            type: instance.config.type,
            status: statusCodeToText(instance.status),
            hostIp: hostIp
          });
        });
      } catch (error) {
        this.instances = [];
        this.$message({ type: "error", message: `错误:${error.message}` });
      } finally {
        this.loading = false;
      }
    },
    handleCurrentChange() {
      this.remoteSelectHandle();
    },
    getServiceByUuid(uuid) {
      this.serviceList.forEach((v) => {
        if (v.uuid === uuid) return v;
      });
    }
  },
  async mounted() {
    await this.renderServices();
  },
  components: {}
};
</script>
