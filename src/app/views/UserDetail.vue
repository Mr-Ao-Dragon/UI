<!--
 * @Author: Copyright 2021 Suwings
 * @Date: 2021-07-03 20:12:26
 * @LastEditTime: 2021-09-02 17:31:50
 * @Description: 
-->

<template>
  <el-row :gutter="20">
    <el-col :span="24">
      <Panel>
        <template #title>基本信息</template>
        <template #default>
          <el-row :gutter="20">
            <el-col :span="12" :offset="0">
              <LineLabel>
                <template #title>唯一标识符</template>
                <template #default>{{ userInfo.uuid }}</template>
              </LineLabel>
              <LineLabel>
                <template #title>用户名</template>
                <template #default>{{ userInfo.userName }}</template>
              </LineLabel>
              <LineLabel>
                <template #title>账号状态</template>
                <template #default>
                  <span style="color: green">{{ moreUserInfo.safe }}</span>
                </template>
              </LineLabel>
            </el-col>
            <el-col :span="12" :offset="0">
              <LineLabel>
                <template #title>注册时间</template>
                <template #default>{{ userInfo.registerTime }}</template>
              </LineLabel>
              <LineLabel>
                <template #title>登录时间</template>
                <template #default>{{ userInfo.loginTime }}</template>
              </LineLabel>
              <LineLabel>
                <template #title>接口</template>
                <template #default>
                  {{ moreUserInfo.api }}
                </template>
              </LineLabel>
            </el-col>
          </el-row>
        </template>
      </Panel>
    </el-col>
  </el-row>
  <el-row :gutter="20">
    <el-col :md="16" :offset="0">
      <Panel>
        <template #title>用户名</template>
        <template #default>
          <div class="sub-title row-mt">
            <p class="sub-title-title">更改用户名</p>
            <p class="sub-title-info">支持中文和字母，长度为 2 到 18 个文字</p>
          </div>
          <el-input
            size="small"
            placeholder=""
            v-model="userData.userName"
            autocomplete="off"
          ></el-input>
          <div style="text-align: right">
            <el-button size="small" class="row-mt" @click="update(1)">更新用户名</el-button>
          </div>
        </template>
      </Panel>
      <Panel>
        <template #title>访问密码</template>
        <template #default>
          <div class="sub-title row-mt">
            <p class="sub-title-title">新密码</p>
            <p class="sub-title-info">长度必须 6 到 18 位，尽可能包含字母数字加符号组合方式</p>
          </div>
          <el-input size="small" type="password" v-model="userData.passWord" :autocomplete="'off'">
          </el-input>
          <div class="sub-title row-mt">
            <p class="sub-title-title">确认新密码</p>
            <p class="sub-title-info">为防止新密码误输入，您必须确认一次新密码</p>
          </div>
          <el-input size="small" type="password" v-model="userData.passWord2" autocomplete="off">
          </el-input>
          <div style="text-align: right">
            <el-button size="small" class="row-mt" @click="update(2)">更新密码</el-button>
          </div>
        </template>
      </Panel>
    </el-col>
    <el-col :md="8" :offset="0">
      <Panel>
        <template #title>注意事项</template>
        <template #default>
          <div class="sub-title">
            <p class="sub-title">关于用户名</p>
            <p class="sub-title-info">
              用户名仅可唯一存在，修改时面板会检查是否存在冲突，如果发现冲突则无法进行修改用户名，用户名请勿包含特殊字符或旧版本系统不兼容的文字。
            </p>
          </div>
          <div class="sub-title">
            <p class="sub-title">关于密码</p>
            <p class="sub-title-info">
              账号密码请尽可能复杂化。请放心，面板管理员无法直接解读您的自定义密码，您的密码即使与其他网站密码有相似之处也不必过于担忧安全问题。
            </p>
          </div>
          <div class="sub-title">
            <p class="sub-title">关于 API 密钥</p>
            <p class="sub-title-info">
              适用于开发者使用的 API 密钥与账号密码登录拥有同等权限，重要程度与密码相同，请勿泄露。
            </p>
          </div>
        </template>
      </Panel>
      <Panel>
        <template #title>API 接口密钥</template>
        <template #default>
          <div class="sub-title row-mt">
            <p class="sub-title-title">什么是 API 密钥？</p>
            <p class="sub-title-info">
              专供给开发者使用的同等权限密钥，可以为您的第三方程序提供充足的功能与可靠的稳定性。
            </p>
          </div>
          <div
            class="row-mt"
            style="
              padding: 12px 8px;
              background: rgba(0, 0, 0, 0.02);
              box-sizing: border-box;
              border-radius: 2px;
              font-size: 13px;
            "
          >
            <span>{{ userInfo.apiKey ? userInfo.apiKey : "未启用" }}</span>
          </div>
          <div style="text-align: right">
            <ItemGroup>
              <el-button size="small" class="row-mt" @click="changeApi(true)"
                >生成 API 密钥</el-button
              >
              <el-button size="small" class="row-mt" @click="changeApi(false)"
                >关闭 API 接口</el-button
              >
            </ItemGroup>
          </div>
        </template>
      </Panel>
    </el-col>
  </el-row>
</template>

<script>
import Panel from "../../components/Panel";
import LineLabel from "../../components/LineLabel";
import { API_URL, API_USER_API, API_USER_UPDATE } from "../service/common";
import { request } from "../service/protocol";

export default {
  data() {
    return {
      userData: {
        userName: "",
        passWord: "",
        passWord2: "",
        apiKey: ""
      },
      moreUserInfo: {
        api: API_URL,
        safe: "账号正常"
      },
      api: {
        enable: false,
        key: ""
      }
    };
  },
  computed: {
    userInfo() {
      return this.$store.state.userInfo;
    }
  },

  async mounted() {},
  methods: {
    async update(type) {
      await this.$confirm("确定要更改此信息吗？", "警告", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      });
      const userName = this.userData.userName;
      const passWord = this.userData.passWord;
      const checkPassWord = this.userData.passWord2;
      if (type === 1) {
        if (userName.length > 18 || userName < 2)
          return this.$message({
            message: "用户名规则不正确，请检查后重试",
            type: "error"
          });
      }
      if (type === 2) {
        if (passWord.length > 18 || passWord.length < 6 || checkPassWord !== passWord)
          return this.$message({
            message: "密码规则不正确或两次密码不一致，请检查后重试",
            type: "error"
          });
      }
      try {
        await request({
          method: "PUT",
          url: API_USER_UPDATE,
          data: {
            userName,
            passWord
          }
        });
        this.$message({ message: "用户数据已更新", type: "success" });
      } catch (error) {
        this.$message({
          message: "用户资料更新失败，可能是用户名冲突或服务器暂时不可用",
          type: "error"
        });
      }
    },

    async changeApi(enable) {
      try {
        const key = await request({
          method: "PUT",
          url: API_USER_API,
          data: {
            enable
          }
        });
        this.$store.commit("setApiKey", key);
        this.$message({ message: "API 操作更改成功", type: "success" });
      } catch (error) {
        this.$message({
          message: `更改失败:${error}`,
          type: "error"
        });
      }
    }
  },
  components: { Panel, LineLabel }
};
</script>