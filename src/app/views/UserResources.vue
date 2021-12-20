<!--
 * @Author: Copyright(c) 2020 Suwings
 * @Date: 2021-05-08 11:53:54
 * @LastEditTime: 2021-12-04 19:23:07
 * @Description: 
-->

<template>
  <Panel>
    <template #title>用户信息</template>
    <template #default>
      <el-row :gutter="20">
        <el-col :span="6" :offset="0">
          <div class="overview-info-warpper">
            <p class="overview-info-title">UUID</p>
            <p class="overview-info-value" v-text="userInfo.uuid"></p>
          </div>
        </el-col>
        <el-col :span="6" :offset="0">
          <div class="overview-info-warpper">
            <p class="overview-info-title">名称</p>
            <p class="overview-info-value" v-text="userInfo.userName"></p>
          </div>
        </el-col>
        <el-col :span="6" :offset="0">
          <div class="overview-info-warpper">
            <p class="overview-info-title">注册时间</p>
            <p class="overview-info-value" v-text="userInfo.registerTime"></p>
          </div>
        </el-col>
        <el-col :span="6" :offset="0">
          <div class="overview-info-warpper">
            <p class="overview-info-title">拥有实例</p>
            <p class="overview-info-value" v-text="userInfo.instances.length"></p>
          </div>
        </el-col>
      </el-row>
    </template>
  </Panel>
  <Panel v-loading="loading">
    <template #title>用户资源管理</template>
    <template #default>
      <div class="sub-title row-mt">
        <p class="sub-title-title">用户资源表</p>
        <p class="sub-title-info">
          当前子用户可管理的所有实例，若实例状态显示“忙碌”代表此实例不存在或远程主机已经离线。
        </p>
      </div>
      <div class="row-mt">
        <ItemGroup>
          <el-button size="small" type="success" @click="openAddInstancePanel">
            <i class="el-icon-plus"></i> 分配实例
          </el-button>
          <el-button size="small" type="primary" @click="save">
            <i class="el-icon-document-checked"></i> 保存数据
          </el-button>
          <el-button size="small" @click="refresh">
            <i class="el-icon-refresh"></i> 刷新
          </el-button>
        </ItemGroup>
      </div>
      <div class="row-mt">
        <el-table :data="userInfo.instances" stripe style="width: 100%" size="small">
          <el-table-column label="远程服务节点">
            <template #default="{ row }"> {{ row.hostIp }}（{{ row.remarks }}） </template>
          </el-table-column>
          <el-table-column prop="nickname" label="实例名称" width="240"></el-table-column>
          <el-table-column label="到期时间">
            <template #default="scope">
              {{ String(scope.row.endTime || "").split("T")[0] }}
            </template>
          </el-table-column>
          <el-table-column prop="status" label="状态">
            <template #default="scope">
              {{ statusToText(scope.row.status) }}
            </template>
          </el-table-column>
          <el-table-column label="操作" style="text-align: center">
            <template #default="scope">
              <el-button size="mini" @click="deleteInstance(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </template>
  </Panel>

  <!-- 新增实例框 -->
  <Dialog v-model="isAddInstanceForUser">
    <template #title>为用户新增实例资源</template>
    <template #default>
      <div class="sub-title">
        <p class="sub-title-title">分配资源</p>
        <p class="sub-title-info">利用远程主机地址与模糊查询来为此用户增加应用实例</p>
      </div>
      <SelectInstance :callback="selectedInstance"></SelectInstance>
    </template>
  </Dialog>
</template>

<script>
import Panel from "../../components/Panel";
import Dialog from "../../components/Dialog";
import SelectInstance from "../../components/SelectInstance";
import { API_USER } from "../service/common";
import { request } from "../service/protocol";
import { statusCodeToText } from "../service/instance_tools";

export default {
  components: { Panel, SelectInstance, Dialog },
  data() {
    return {
      userUuid: this.$route.params.userUuid,
      userInfo: {
        instances: []
      },
      isAddInstanceForUser: false,
      loading: true
    };
  },
  methods: {
    openAddInstancePanel() {
      this.isAddInstanceForUser = true;
    },
    selectedInstance(row) {
      this.userInfo.instances.push({
        instanceUuid: row.instanceUuid,
        serviceUuid: row.serviceUuid,
        nickname: row.nickname,
        status: row.status,
        hostIp: row.hostIp
      });
      this.isAddInstanceForUser = false;
    },
    deleteInstance(row) {
      this.userInfo.instances.forEach((v, index) => {
        if (v.instanceUuid === row.instanceUuid) {
          this.userInfo.instances.splice(index, 1);
        }
      });
    },
    statusToText(code) {
      return statusCodeToText(code);
    },
    async save() {
      try {
        await request({
          method: "PUT",
          url: API_USER,
          data: {
            uuid: this.userUuid,
            config: this.userInfo
          }
        });
        this.$message({ type: "success", message: "更新成功" });
      } catch (error) {
        this.$message({
          type: "error",
          message: `错误: ${error.message}`
        });
      }
    },
    // 用户数据渲染
    async render() {
      const result = await request({
        method: "GET",
        url: API_USER,
        params: {
          uuid: this.userUuid,
          advanced: true
        }
      });
      this.userInfo = result;
      this.loading = false;
    },
    async refresh() {
      await this.render();
      this.$message({ type: "info", message: "已刷新" });
    }
  },
  async mounted() {
    await this.render();
  }
};
</script>