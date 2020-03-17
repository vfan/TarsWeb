<template>
  <div>
    <div style="width:600px">
      <el-form @submit.native.prevent="save">
        <el-form-item :label="$t('user.op')">
          <el-input
            type="textarea"
            v-model="operator"
            :disabled="!hasAuth"
          ></el-input>
        </el-form-item>
        <el-form-item :label="$t('user.dev')">
          <el-input
            type="textarea"
            v-model="developer"
            :disabled="!hasAuth"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button
            @click="save"
            type="primary"
            theme="primary"
            v-if="hasAuth"
            >{{ $t("common.submit") }}</el-button
          >
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: "ServerUserManage",

  data() {
    return {
      serverData: Object.assign({}, this.$parent.getServerData()),
      developer: "",
      operator: "",
      hasAuth: false
    };
  },

  mounted() {
    this.checkHasAuth();
    this.getAuthList();
  },

  methods: {
    checkHasAuth() {
      this.$ajax
        .getJSON("/server/api/has_auth", {
          application: this.serverData.application,
          server_name: this.serverData.server_name,
          role: "developer"
        })
        .then(data => {
          console.log("checkHasAuth", data);
          this.hasAuth = data.has_auth || false;
        })
        .catch(err => {
          this.$tip.error(
            `${this.$t("common.error")}: ${err.message || err.err_msg}`
          );
        });
    },
    getAuthList() {
      this.$ajax
        .getJSON("/server/api/get_auth_list", {
          application: this.serverData.application,
          server_name: this.serverData.server_name
        })
        .then(data => {
          this.operator = (data.operator || []).join(";");
          this.developer = (data.developer || []).join(";");
        })
        .catch(err => {
          this.$tip.error(
            `${this.$t("common.error")}: ${err.message || err.err_msg}`
          );
        });
    },
    save() {
      const loading = this.$Loading.show();
      this.$ajax
        .postJSON("/server/api/update_auth", {
          application: this.serverData.application,
          server_name: this.serverData.server_name,
          operator: this.operator,
          developer: this.developer
        })
        .then(data => {
          loading.hide();
          this.$tip.success(`${this.$t("common.success")}`);
        })
        .catch(err => {
          loading.hide();
          this.$tip.error(
            `${this.$t("common.error")}: ${err.message || err.err_msg}`
          );
        });
    }
  }
};
</script>

<style>
.page_server_server_monitor {
  padding-bottom: 20px;

  .charts {
    margin-top: 20px;
  }

  .hours-filter {
    margin-bottom: 16px;
  }
}
</style>
