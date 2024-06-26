<!--
  Copyright (C) 2022 Suwings(https://github.com/Suwings)

  This program is free software: you can redistribute it and/or modify
  it under the terms of the GNU General Public License as published by
  the Free Software Foundation, either version 3 of the License, or
  (at your option) any later version.

  This program is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  GNU General Public License for more details.
  
  According to the GPL, it is forbidden to delete all copyright notices, 
  and if you modify the source code, you must open source the
  modified source code.

  版权所有 (C) 2022 Suwings(https://github.com/Suwings)

  本程序为自由软件，你可以依据 GPL 的条款（第三版或者更高），再分发和/或修改它。
  该程序以具有实际用途为目的发布，但是并不包含任何担保，
  也不包含基于特定商用或健康用途的默认担保。具体细节请查看 GPL 协议。

  根据协议，您必须保留所有版权声明，如果修改源码则必须开源修改后的源码。
  前往 https://mcsmanager.com/ 申请闭源开发授权或了解更多。
-->

<template>
  <Panel>
    <template #title>分布式应用实例列表</template>
    <template #default>
      <el-row :gutter="20" justify="space-between" class="row-mb">
        <el-col :md="12" :offset="0">
          <ItemGroup>
            <el-select
              style="width: 320px"
              v-model="currentRemoteUuid"
              filterable
              placeholder="请选择远程服务地址"
              size="small"
              @change="remoteSelectHandle"
            >
              <el-option
                v-for="item in remoteList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
            <el-input
              v-model="query.instanceName"
              placeholder="实例名称"
              size="small"
              style="width: 160px"
            ></el-input>
            <el-button size="small" @click="refresh">
              <i class="el-icon-refresh"></i> 刷新
            </el-button>
          </ItemGroup>
        </el-col>
        <el-col :md="12" :offset="0">
          <ItemGroup style="text-align: right">
            <el-button size="small" type="success" @click="toNewInstance">
              <i class="el-icon-plus"></i> 新建实例
            </el-button>
            <el-button size="small" @click="batOpen">
              <i class="el-icon-video-play"></i> 开启
            </el-button>
            <el-button size="small" @click="batStop">
              <i class="el-icon-video-pause"></i> 关闭
            </el-button>
            <el-button size="small" @click="batKill">
              <i class="el-icon-video-pause"></i> 终止
            </el-button>
            <el-button size="small" type="danger" plain @click="batDelete(1)">
              <i class="el-icon-delete"></i> 移除
            </el-button>
            <el-button size="small" type="danger" @click="batDelete(2)">
              <i class="el-icon-delete"></i> 删除
            </el-button>
          </ItemGroup>
        </el-col>
      </el-row>

      <div>
        <div class="instance-table-warpper">
          <div>
            <p class="color-red" v-if="!currentRemoteUuid">
              未选择任何远程服务，请通过这里选择一个地址。若没有任何可选项，请前往“分布式服务”界面进行设置
            </p>
          </div>
          <div>
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
        </div>

        <el-table
          :data="instances"
          stripe
          style="width: 100%"
          size="mini"
          ref="multipleTable"
          @selection-change="selectionChange"
        >
          <el-table-column type="selection" width="55"> </el-table-column>
          <el-table-column prop="nickname" label="实例昵称" min-width="240"></el-table-column>
          <el-table-column prop="status" label="运行状态" width="120">
            <template #default="scope">
              <div class="color-gray" v-if="scope.row.status == 0">
                <i class="el-icon-video-pause"></i>
                <span> 未运行</span>
              </div>
              <div class="color-green" v-else-if="scope.row.status == 3">
                <i class="el-icon-video-play"></i>
                <span> 运行中</span>
              </div>
              <span class="color-yellow" v-else-if="scope.row.status == 1">停止中</span>
              <span class="color-yellow" v-else-if="scope.row.status == 2">启动中</span>

              <span class="color-red" v-else-if="scope.row.status == -1">忙碌</span>
              <span class="color-red" v-else>忙碌</span>
            </template>
          </el-table-column>
          <el-table-column prop="type" label="实例类型" width="140"></el-table-column>
          <el-table-column label="操作" style="text-align: center" width="180">
            <template #default="scope">
              <el-button
                size="mini"
                @click="editInstance(scope.row.serviceUuid, scope.row.instanceUuid)"
              >
                设置
              </el-button>
              <el-button
                size="mini"
                @click="toInstance(scope.row.serviceUuid, scope.row.instanceUuid)"
              >
                管理
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </template>
  </Panel>
</template>

<script>
import { CircleCheckFilled, CircleCloseFilled } from "@element-plus/icons";
import Panel from "../../components/Panel";
// import LineLabel from "../../components/LineLabel";
import { ElMessage } from "element-plus";
import axios from "axios";
import { API_INSTANCE, API_SERVICE_INSTANCES, API_SERVICE_LIST, API_URL } from "../service/common";
import router from "../router";
import { request } from "../service/protocol";
import { typeTextToReadableText } from "../service/instance_tools";
export default {
  // eslint-disable-next-line vue/no-unused-components
  components: { Panel, CircleCheckFilled, CircleCloseFilled },
  data() {
    return {
      remoteList: [],
      currentRemoteUuid: null,
      instances: [],
      multipleSelection: [], // 表格多选属性
      startedInstance: 0,
      loading: true,
      availableService: [], // 可用和不可用远程服务列表
      unavailableService: [],

      page: 1,
      maxPage: 1,

      query: {
        instanceName: ""
      }
    };
  },
  async mounted() {
    await this.render();
  },
  beforeUnmount() {},
  methods: {
    // 获取分布式服务列表（不包括具体实例列表）
    async displayRemoteServiceList() {
      const data = await request({
        method: "GET",
        url: API_SERVICE_LIST
      });
      for (const service of data) {
        const ip = `${service.ip}:${service.port}`;
        const remarks = `${service.remarks}`;
        if (service.available) {
          this.remoteList.push({
            value: service.uuid,
            label: `${ip} ${remarks}`
          });
          this.availableService.push(service);
        } else {
          this.remoteList.push({
            value: service.uuid,
            label: `${ip} ${remarks} (离线)`
          });
          this.unavailableService.push(service);
        }
      }

      // 如果存在上次的选择记录，那么直接跳转到上次记录
      const lastSelected = localStorage.getItem("pageSelectedRemoteUuid");
      if (lastSelected) {
        this.remoteList.forEach((v) => {
          if (v.value === lastSelected) {
            this.currentRemoteUuid = v.value;
            return this.remoteSelectHandle();
          }
        });
      }
    },
    // 获取分布式服务具体实例列表
    async remoteSelectHandle() {
      try {
        if (!this.currentRemoteUuid) throw new Error("还未选择远程服务器");
        this.startedInstance = 0;
        this.instances = [];
        this.loading = true;
        const result = await request({
          method: "GET",
          url: API_SERVICE_INSTANCES,
          params: {
            remote_uuid: this.currentRemoteUuid,
            page: this.page,
            page_size: 20,
            instance_name: this.query.instanceName
          }
        });
        // 页码调整
        this.page = result.page;
        this.maxPage = result.maxPage;
        const instances = result.data;
        instances.forEach((instance) => {
          const status = instance.status;
          const type = typeTextToReadableText(instance.config.type);
          // 计算正在运行的实例
          if (instance.status != 0) this.startedInstance++;
          // 压入所有实例
          this.instances.push({
            instanceUuid: instance.instanceUuid,
            serviceUuid: this.currentRemoteUuid,
            nickname: instance.config.nickname,
            type,
            status
          });
        });
        this.loading = false;
        // 记录当前选择的远程服务，方便下次直接加载
        localStorage.setItem("pageSelectedRemoteUuid", this.currentRemoteUuid);
      } catch (error) {
        this.$notify({
          title: "访问远程服务异常",
          message: error.toString(),
          type: "error"
        });
      }
    },
    // 分页改变
    handleCurrentChange() {
      this.refresh();
    },
    refresh() {
      this.remoteSelectHandle();
    },
    async render() {
      await this.displayRemoteServiceList();
    },
    // 表格多选函数
    selectionChange(v) {
      if (v.length == 0) this.canInterval = true;
      else this.canInterval = false;
      this.multipleSelection = v;
    },
    editInstance(serviceUuid, instanceUuid) {
      console.log("编辑实例:", serviceUuid, instanceUuid);
      router.push({ path: `/instance_detail/${serviceUuid}/${instanceUuid}/` });
    },
    toNewInstance() {
      if (!this.currentRemoteUuid) {
        return this.$message({ type: "info", message: "请先在左侧下拉框中选择远程服务" });
      }
      router.push({ path: `/new_instace/${this.currentRemoteUuid}` });
    },
    toInstance(serviceUuid, instanceUuid) {
      console.log("访问实例:", serviceUuid, instanceUuid);
      router.push({ path: `/terminal/${serviceUuid}/${instanceUuid}/` });
    },
    // 批量删除
    async batDelete(type) {
      if (type === 1) {
        await this.$confirm(
          "确定要进行批量移除嘛？此操作不会删除实例实际文件，只会删除实例",
          "最终确认",
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }
        );
      } else {
        await this.$confirm("确定要进行批量删除吗？此操作将会一并删除文件", "最终确认", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        });
      }

      const uuids = [];
      for (const iterator of this.multipleSelection) {
        uuids.push(iterator.instanceUuid);
      }
      if (uuids.length === 0) {
        return this.$message({ type: "error", message: "请至少选择一项" });
      }
      await axios.request({
        method: "DELETE",
        url: API_INSTANCE,
        params: {
          remote_uuid: this.currentRemoteUuid
        },
        data: { uuids, deleteFile: type === 1 ? false : true }
      });
      this.$notify({
        title: "批量删除成功",
        message: "可能会存在一定延迟，文件删除需要一定的时间"
      });
    },
    async batKill() {
      if (this.multipleSelection.length == 0)
        return ElMessage.error("无法执行，请至少选择一个实例");
      await axios.request({
        method: "POST",
        url: `${API_URL}/api/instance/multi_kill/`,
        data: this.multipleSelection
      });
      this.$notify({
        title: "终止命令已发出",
        message: "已成功向各个远程主机发布命令，具体操作可能略有延时，请稍等一段时间后查看结果"
      });
    },
    async batOpen() {
      if (this.multipleSelection.length == 0)
        return ElMessage.error("无法执行，请至少选择一个实例");
      await axios.request({
        method: "POST",
        url: `${API_URL}/api/instance/multi_open/`,
        data: this.multipleSelection
      });
      this.$notify({
        title: "开启命令已发出",
        message: "已成功向各个远程主机发布命令，具体操作可能略有延时，请稍等一段时间后查看结果"
      });
    },
    async batStop() {
      if (this.multipleSelection.length == 0)
        return ElMessage.error("无法执行，请至少选择一个实例");
      await axios.request({
        method: "POST",
        url: `${API_URL}/api/instance/multi_stop/`,
        data: this.multipleSelection
      });
      this.$notify({
        title: "关闭命令已发出",
        message: "已成功向各个远程主机发布命令，具体操作可能略有延时，请稍等一段时间后查看结果"
      });
    }
  }
};
</script>
