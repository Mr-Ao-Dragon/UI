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
  <el-row :gutter="20">
    <el-col :span="24">
      <Panel v-loading="loading">
        <template #title>面板端系统数据</template>
        <template #default>
          <el-row :gutter="20">
            <el-col :xs="12" :md="6" v-for="(item, index) in computerInfoA" :key="index">
              <div class="overview-info-warpper">
                <p class="overview-info-title" v-html="item.name"></p>
                <p
                  class="overview-info-value"
                  v-text="item.value"
                  :class="{ 'color-red': item.warn }"
                ></p>
              </div>
            </el-col>

            <el-col :span="24" :offset="0">
              <div class="box-card-title-more">面板总览数据</div>
            </el-col>

            <el-col :xs="12" :md="6" v-for="(item, index) in computerInfoB" :key="index">
              <div class="overview-info-warpper">
                <p class="overview-info-title" v-html="item.name"></p>
                <p
                  class="overview-info-value"
                  v-text="item.value"
                  :class="{ 'color-red': item.warn }"
                ></p>
              </div>
            </el-col>
          </el-row>
        </template>
      </Panel>
      <el-row :gutter="20">
        <el-col :md="12" :offset="0">
          <Panel v-loading="loading">
            <template #title>面板端接口请求量</template>
            <template #default>
              <p>每1分钟统计间隔，总计1小时的请求历史</p>
              <div class="echart-wrapper">
                <div id="echart-wrapper-main3" style="width: 100%; height: 200px"></div>
              </div>
            </template>
          </Panel>
        </el-col>
        <el-col :md="12" :offset="0">
          <Panel v-loading="loading">
            <template #title>面板端 CPU 使用率</template>
            <template #default>
              <p>每10秒统计间隔，总十分钟的 CPU 历史使用率</p>
              <div class="echart-wrapper">
                <div id="echart-wrapper-main" style="width: 100%; height: 200px"></div>
              </div>
            </template>
          </Panel>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :md="12" :offset="0">
          <Panel v-loading="loading">
            <template #title>分布式应用实例状态</template>
            <template #default>
              <p>每1分钟统计间隔，总计1小时的实例状态历史</p>
              <div class="echart-wrapper">
                <div id="echart-wrapper-main4" style="width: 100%; height: 200px"></div>
              </div>
            </template>
          </Panel>
        </el-col>
        <el-col :md="12" :offset="0">
          <Panel v-loading="loading">
            <template #title>面板端内存使用率</template>
            <template #default>
              <p>每10秒统计间隔，总十分钟的内存历史使用率</p>
              <div class="echart-wrapper">
                <div id="echart-wrapper-main2" style="width: 100%; height: 200px"></div>
              </div>
            </template>
          </Panel>
        </el-col>
      </el-row>

      <Panel v-loading="loading">
        <template #title>分布式服务总览</template>
        <template #default>
          <p>
            确保所有远程服务均在线，离线状态将导致此远程服务以及相关功能不可用，可能会影响使用体验与数据。
          </p>
          <el-table :data="servicesStatus" style="width: 100%" size="small">
            <el-table-column prop="ip" label="地址" width="180"> </el-table-column>
            <el-table-column prop="port" label="端口" width="180"> </el-table-column>
            <el-table-column prop="cpu" label="CPU"> </el-table-column>
            <el-table-column prop="mem" label="内存"> </el-table-column>
            <el-table-column prop="instance" label="已有实例"> </el-table-column>
            <el-table-column prop="started" label="运行实例"> </el-table-column>
            <el-table-column prop="status" label="连接状态"> </el-table-column>
          </el-table>
        </template>
      </Panel>
    </el-col>
  </el-row>

  <Panel v-if="manualLink">
    <template #title>帮助文档</template>
    <template #default>
      <el-row :gutter="20">
        <el-col :md="6" :offset="0" v-for="(item, index) in manualLink['helpLink']" :key="index">
          <a class="manualLink" :href="item.link" v-text="item.title" target="_black"></a>
        </el-col>

        <el-col :span="24">
          <div class="box-card-title-more">常见问题</div>
        </el-col>
        <el-col :md="6" :offset="0" v-for="(item, index) in manualLink['faq']" :key="index">
          <a class="manualLink" :href="item.link" v-text="item.title" target="_black"></a>
        </el-col>
      </el-row>
    </template>
  </Panel>
</template>

<script>
import * as echarts from "echarts";
import Panel from "../../components/Panel";
// import LineLabel from "../../components/LineLabel";
import { request } from "../service/protocol";
import { API_OVERVIEW } from "../service/common";
import {
  getDefaultOption,
  getStatusChartOption1,
  getStatusChartOption2
} from "../service/chart_option";
export default {
  data: function () {
    return {
      loading: true,

      systemChart: null,
      systemChart2: null,
      requestChart: null,
      systemChartData: [],
      data: {},
      interval: null,
      computerInfoA: [],
      computerInfoB: [],
      servicesStatus: [],
      manualLink: null
    };
  },
  methods: {
    startInterval() {
      this.interval = setInterval(async () => {
        const data = await this.request();
        this.render(data);
        this.setChartSource();
      }, 5000);
    },
    stopInterval() {
      clearInterval(this.interval);
    },
    async request() {
      return await request({
        method: "GET",
        url: API_OVERVIEW
      });
    },
    render(data) {
      const system = data.system;
      // 表格数据渲染
      if (data.chart) this.systemChartData = data.chart;
      const remoteCount = data.remoteCount;
      const remote = data.remote;
      // 计算总实例与运行实例数
      let totalInstance = 0;
      let runningInstance = 0;
      for (const iterator of remote) {
        if (iterator.instance) {
          totalInstance += iterator.instance.total;
          runningInstance += iterator.instance.running;
        }
      }
      // 计算内存
      const free = Number(system.freemem / 1024 / 1024 / 1024).toFixed(1);
      const total = Number(system.totalmem / 1024 / 1024 / 1024).toFixed(1);
      const used = Number(total - free).toFixed(1);
      // 计算已正常运行时间
      // const uptime = Number(system.uptime / 60 / 60).toFixed(0);
      this.computerInfoA = [
        {
          name: "系统类型",
          value: `${system.type} ${system.platform}`
        },
        {
          name: "系统版本",
          value: `${system.version} ${system.release}`
        },
        {
          name: "面板端时间",
          value: system.time
        },
        {
          name: "本地时间",
          value: new Date().toLocaleDateString() + " " + new Date().toLocaleTimeString()
        },
        {
          name: "计算机名称",
          value: system.hostname
        },
        {
          name: "进程权限用户",
          value: system.user.username
        },
        {
          name: "内存使用",
          value: `${used}GB/${total}GB`,
          warn: used / total > 0.9
        },
        {
          name: "系统 CPU 使用率",
          value: `${Number(system.cpu * 100).toFixed(1)}%`,
          warn: system.cpu * 100 > 90
        }
      ];
      this.computerInfoB = [
        {
          name: "Node 版本",
          value: system.node
        },
        {
          name: "面板版本",
          value: data.version
        },
        {
          name: "分布式在线",
          value: `${remoteCount.available}/${remoteCount.total}`,
          warn: remoteCount.available !== remoteCount.total
        },
        {
          name: "实例运行数",
          value: `${runningInstance}/${totalInstance}`
        },

        {
          name: "面板已用内存",
          value: `${Number(data.process.memory / 1024 / 1024).toFixed(0)}MB`
        },
        {
          name: "越权请求次数",
          value: data.record.illegalAccess
        },
        {
          name: "登录失败与总次数",
          value: `${data.record.loginFailed}/${data.record.logined}`
        },
        {
          name: "封禁 IP 数",
          value: data.record.banips,
          warn: data.record.banips > 0
        }
      ];
      // 装载远程服务信息
      this.servicesStatus = [];
      for (const iterator of remote) {
        if (iterator.system) {
          const cpu = Number(iterator.system.cpuUsage * 100).toFixed(1);
          const totalmem = Number(iterator.system.totalmem / 1024 / 1024 / 1024).toFixed(1);
          const freemem = Number(iterator.system.freemem / 1024 / 1024 / 1024).toFixed(1);
          const usedmem = Number(totalmem - freemem).toFixed(1);
          this.servicesStatus.push({
            ip: iterator.ip,
            port: iterator.port,
            cpu: `${cpu}%`,
            mem: `${usedmem}GB/${totalmem}GB`,
            instance: iterator.instance.total,
            started: iterator.instance.running,
            status: iterator.available ? "正常" : "离线"
          });
        } else {
          this.servicesStatus.push({
            ip: iterator.ip,
            port: iterator.port,
            cpu: "--",
            mem: "--",
            instance: "--",
            started: "--",
            status: iterator.available ? "正常" : "离线"
          });
        }
      }
    },
    initChart() {
      console.log("正在初始化表格....");
      // 基于准备好的dom，初始化echarts实例
      this.systemChart = echarts.init(document.getElementById("echart-wrapper-main"));
      this.systemChart.setOption(getDefaultOption());
      // this.systemChart.resize({});
      this.systemChart2 = echarts.init(document.getElementById("echart-wrapper-main2"));
      this.systemChart2.setOption(getDefaultOption());
      this.systemChart3 = echarts.init(document.getElementById("echart-wrapper-main3"));
      this.systemChart3.setOption(getStatusChartOption1());
      this.systemChart4 = echarts.init(document.getElementById("echart-wrapper-main4"));
      this.systemChart4.setOption(getStatusChartOption2());
      this.setChartSource();
    },
    setRequestChart() {
      const MAX_TIME = this.systemChartData["request"].length - 1;
      const source = this.systemChartData["request"];
      for (const key in source) {
        source[key]["time"] = `${MAX_TIME - key * 1} 分前`;
      }
      this.systemChart3.setOption({
        dataset: {
          dimensions: ["time", "value"],
          source
        }
      });
      this.systemChart4.setOption({
        dataset: {
          dimensions: ["time", "totalInstance", "runningInstance"],
          source
        }
      });
    },
    setSystemChart() {
      const MAX_TIME = this.systemChartData["system"].length - 1;
      const source = this.systemChartData["system"];
      for (const key in source) {
        source[key]["time"] = `${(MAX_TIME - key) * 10} 秒前`;
      }
      this.systemChart.setOption({
        dataset: {
          dimensions: ["time", "cpu"],
          source
        }
      });
      this.systemChart2.setOption({
        dataset: {
          dimensions: ["time", "mem"],
          source
        }
      });
    },
    setChartSource() {
      this.setRequestChart();
      this.setSystemChart();
    }
  },
  components: { Panel },
  async mounted() {
    this.loading = true;
    const data = await this.request();
    this.render(data);
    this.initChart();
    this.loading = false;
    this.manualLink = window.onlineMCSManagerNotice ? window.onlineMCSManagerNotice() : null;
    this.startInterval();
  },
  beforeUnmount() {
    this.stopInterval();
    this.systemChart.dispose();
    this.systemChart2.dispose();
    this.systemChart3.dispose();
    this.systemChart4.dispose();
  }
};
</script>
