<template>
  <el-row>
    <el-row>
      <el-form label-position="right">
        <el-form-item label="学校名称">
          <el-input v-model="name" placeholder="School Name" style="width:200px"></el-input>
          <el-input v-model="ojname" placeholder="OJ Name" style="width:200px"></el-input>
        </el-form-item>
        <el-form-item label="是否开启WIKI和TodoList（用于比赛时防止查阅资料）">
          <el-switch v-model="wikiopen" active-text="开启" inactive-text="关闭"></el-switch>
        </el-form-item>
        <el-form-item label="开启语言（中间用 | 隔开，确保语言在判题机中支持！）">
          <el-input v-model="openlanguage" placeholder="中间用 | 隔开，确保语言在判题机中支持！" style="width:300px"></el-input>
        </el-form-item>
        <el-form-item label="开启OI模式（样例全判）">
          <el-switch v-model="openoi" active-text="开启" inactive-text="关闭"></el-switch>
        </el-form-item>
        <el-form-item label="是否开启源码查看（除管理员外不得查看源码，自己只能看自己的代码）">
          <el-switch v-model="openstatus" active-text="开启" inactive-text="关闭"></el-switch>
        </el-form-item>

        <el-form-item label="是否游客访问（关闭后，必须登录才能访问到OJ内容）">
          <el-switch v-model="openvisitor" active-text="开启" inactive-text="关闭"></el-switch>
        </el-form-item>

        <el-form-item label="是否开放注册">
          <el-switch v-model="openregister" active-text="开启" inactive-text="关闭"></el-switch>
        </el-form-item>
        <el-form-item label="是否允许自己查看自己的代码">
          <el-switch v-model="openselfstatus" active-text="开启" inactive-text="关闭"></el-switch>
        </el-form-item>
        <el-button style="margin-top:20px;" type="primary" @click="click">提交</el-button>
      </el-form>
      <br>
      <h3>Banner设置</h3>
      <el-table :data="tableData" size="small">
        <el-table-column prop="msg" label="Banner"></el-table-column>
        <el-table-column prop="time" label="Time"></el-table-column>
        <el-table-column fixed="right" label="操作" width="160">
          <template slot-scope="scope">
            <el-button @click="delmsg(scope.row)" type="danger" size="small">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <br>
      <el-input v-model="msg" placeholder="请输入Banner，支持Html" style="width:400px"></el-input>
      <el-button type="primary" @click="sendbanner">提交</el-button>
    </el-row>
  </el-row>
</template>

<script>
export default {
  name: "adminsetting",
  data() {
    return {
      name: "",
      ojname: "GenchOJ",
      wikiopen: true,
      tableData: [],
      msg: "",
      openlanguage:"C++|C|Python3|Swift5.1|Java",
      openoi:true,
      openstatus:true,
      openvisitor:true,
      openregister:true,
      openselfstatus:true,
    };
  },
  methods: {
    sendbanner() {
      this.$axios
        .post("/banner/", { msg: this.msg })
        .then(res => {
          this.$message.success("提交成功！");
          this.reflash();
        })
        .catch(error => {
          this.$message.error(
            "服务器错误！" + "(" + JSON.stringify(error.response.data) + ")"
          );
        });
    },
    reflash() {
      this.$axios
        .get("/banner/")
        .then(res => {
          this.tableData = res.data;
        })
        .catch(error => {
          this.$message.error(
            "服务器错误！" + "(" + JSON.stringify(error.response.data) + ")"
          );
        });
    },
    delmsg(row) {
      this.$axios.delete("/banner/" + row.id + "/").then(response => {
        this.$message.success("删除成功！");
        this.reflash();
      });
    },
    click() {
      if (this.name == "无") {
        this.$axios
          .post("/settingboard/", {
            schoolname: this.name,
            ojname: this.ojname,
            openwiki: this.wikiopen,
            openlanguage:this.openlanguage,
            openoi:this.openoi,
            openstatus:this.openstatus,
            openvisitor:this.openvisitor,
            openregister:this.openregister,
            openselfstatus:this.openselfstatus,
          })
          .then(res => {
            this.$message.success("提交成功！");
          })
          .catch(error => {
            this.$message.error(
              "服务器错误！" + "(" + JSON.stringify(error.response.data) + ")"
            );
          });
      } else {
        this.$axios
          .put("/settingboard/1/", {
            schoolname: this.name,
            ojname: this.ojname,
            openwiki: this.wikiopen,
            openlanguage:this.openlanguage,
            openoi:this.openoi,
            openstatus:this.openstatus,
            openvisitor:this.openvisitor,
            openregister:this.openregister,
            openselfstatus:this.openselfstatus,
          })
          .then(res => {
            this.$message.success("提交成功！");
          })
          .catch(error => {
            this.$message.error(
              "服务器错误！" + "(" + JSON.stringify(error.response.data) + ")"
            );
          });
      }
    }
  },
  created() {
    this.$axios
      .get("/settingboard/")
      .then(res => {
        if (res.data.length > 0) {
          this.name = res.data[0].schoolname;
          this.ojname = res.data[0].ojname;
          this.wikiopen = res.data[0].openwiki;
          this.openlanguage = res.data[0].openlanguage;
          this.openoi = res.data[0].openoi;
          this.openstatus = res.data[0].openstatus;
          this.openvisitor =res.data[0].openvisitor;
          this.openregister =res.data[0].openregister;
          this.openselfstatus= res.data[0].openselfstatus;
        } else this.name = "无";
        this.reflash();
      })
      .catch(error => {
        this.$message.error(
          "服务器错误！" + "(" + JSON.stringify(error.response.data) + ")"
        );
      });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-tag + .el-tag {
  margin-left: 10px;
}
</style>
