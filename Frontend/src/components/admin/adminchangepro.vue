<template>
  <el-row>
    <el-input placeholder="输入Title或Tag来筛选..."
              v-model="searchpro"
              @keyup.native.enter="searchtitle"
              style="float:right;width:300px;">
      <el-button slot="append"
                 icon="el-icon-search"
                 @click="searchtitle"></el-button>
    </el-input>
    <el-pagination @current-change="handleCurrentChange"
                   :current-page="currentpage"
                   :page-size="20"
                   :total="totalproblem"
                   layout="total,prev, pager, next, jumper"></el-pagination>

    <el-table :data="gridData">
      <el-table-column property="problem"
                       label="ID"
                       width="70"></el-table-column>
      <el-table-column property="title"
                       label="标题"
                       width="350"></el-table-column>
      <el-table-column property="tag"
                       label="标签"></el-table-column>
      <el-table-column property="score"
                       label="分数"
                       width="80"></el-table-column>
      <el-table-column property="oj"
                       label="OJ"
                       width="70"></el-table-column>
      <el-table-column property="level"
                       label="难度"
                       width="70"></el-table-column>
      <el-table-column property="ac"
                       label="AC数"
                       width="70"></el-table-column>
      <el-table-column property="submission"
                       label="提交数"
                       width="70"></el-table-column>
      <el-table-column fixed="right"
                       label="操作"
                       width="240">
        <template slot-scope="scope">
          <el-button @click="handleEdit(scope.row)"
                     type="primary"
                     size="small">编辑</el-button>
          <el-button @click="problemclick(scope.row)"
                     type="primary"
                     size="small">查看</el-button>
          <el-button @click="handleDel(scope.row)"
                     type="danger"
                     size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog title="修改题目"
               :visible.sync="dialogTableVisible"
               width="85%">
      <el-form ref="problemform"
               :model="problemform"
               label-position="right"
               v-loading="loading">
        <el-form-item label="题目编号：">
          <el-input style="width:200px;"
                    v-model="problemform.problem"
                    placeholder="请输入题目编号"
                    @change="problemchange"
                    readonly>
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="problemchange"></el-button>
          </el-input>
        </el-form-item>

        <el-form-item label="特殊选项：添加其他OJ题目用！不知道的话请忽略">
          <el-input v-model="problemform.oj"
                    placeholder="OJ"
                    style="width:100px;"></el-input>
          <el-input v-model="problemform.source"
                    placeholder="Pro ID"
                    style="width:100px;margin-left:40px;"></el-input>
        </el-form-item>
        <el-form-item label="作者：">
          <el-input v-model="problemform.author"
                    style="width:400px;"></el-input>
        </el-form-item>
        <el-form-item label="标题：">
          <el-input v-model="problemform.title"
                    style="width:400px;"></el-input>
        </el-form-item>
        <el-form-item label="介绍：">
          <el-input type="textarea"
                    v-model="problemform.des"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="输入：">
          <el-input type="textarea"
                    v-model="problemform.input"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="输出：">
          <el-input type="textarea"
                    v-model="problemform.output"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="样例输入（用 |#) 分割）：">
          <el-input type="textarea"
                    v-model="problemform.sinput"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="样例输出（用 |#) 分割）：">
          <el-input type="textarea"
                    v-model="problemform.soutput"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="提示：">
          <el-input type="textarea"
                    v-model="problemform.hint"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="来源：">
          <el-input v-model="problemform.source"
                    style="width:400px;"></el-input>
        </el-form-item>
        <el-form-item label="模板代码：（用*****作为语言分割，如 *****C++***** xxxx *****C***** xxxx *****Python2***** xxxxx）">
          <el-input type="textarea"
                    v-model="problemform.template"
                    autosize
                    style="width:800px;"></el-input>
        </el-form-item>
        <el-form-item label="时间（ms）：">
          <el-input-number style="width:200px;"
                           v-model="problemform.time"
                           :step="1000"
                           :min="100"
                           :max="60000"></el-input-number>
        </el-form-item>
        <el-form-item label="内存（MB）：">
          <el-input-number style="width:200px;"
                           v-model="problemform.memory"
                           :step="64"
                           :min="4"
                           :max="1024"></el-input-number>
        </el-form-item>
        <el-form-item label="权限：">
          <el-select v-model="problemform.auth"
                     placeholder="请选择"
                     style="width:200px;">
            <el-option key="1"
                       label="公开"
                       :value="1"></el-option>
            <el-option key="2"
                       label="私密"
                       :value="2"></el-option>
            <el-option key="3"
                       label="比赛中"
                       :value="3"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="难度：">
          <el-select v-model="problemform.level"
                     placeholder="请选择"
                     style="width:200px;">
            <el-option key="1"
                       label="简单"
                       :value="1"></el-option>
            <el-option key="2"
                       label="普通"
                       :value="2"></el-option>
            <el-option key="3"
                       label="中等"
                       :value="3"></el-option>
            <el-option key="4"
                       label="困难"
                       :value="4"></el-option>
            <el-option key="5"
                       label="极其困难"
                       :value="5"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="标签（用|分割）：">
          <el-input v-model="problemform.tag"
                    style="width:400px;"></el-input>
        </el-form-item>
        <el-form-item label="分数（建议100~10000）：">
          <el-input-number style="width:200px;"
                           v-model="problemform.score"
                           :step="100"
                           :min="100"
                           :max="10000"></el-input-number>
        </el-form-item>

        <el-upload style="width:400px;"
                   ref="upload"
                   :action="uploadaddress"
                   :on-exceed="handleExceed"
                   :on-change="handleChange"
                   :on-success="handleSuccess"
                   :on-error="handleError"
                   :on-remove="handleRemove"
                   :file-list="fileList"
                   :multiple="false"
                   :limit="1"
                   :auto-upload="false"
                   :http-request="myupload">
          <el-button slot="trigger"
                     size="small"
                     type="primary">选取数据文件</el-button>
          <div slot="tip"
               class="el-upload__tip">只能上传zip/jpg文件【注意是小写字母后缀】,压缩包内的不要有文件夹，输入输出文件后缀为.in和.out.添加一个casedes.txt文件（utf-8编码）可以对每一个样例进行说明，每行一个说明，中间不要有多余的空行，对应的case用|隔开，如： case1|这是case1的说明</div>
        </el-upload>
        <h5><a style="text-decoration: none;color:#67C23A;" target="_blank" href="https://docs.lpoj.cn/doc/judger.html#special-judge">点我看判题机文档</a></h5>


        <el-form-item>
          <el-input v-model="serviceaddress"
                    placeholder="下载数据需要填写服务端地址，如：http://localhost:8000 ,注意最后不要有斜杠（临时功能）"></el-input>
          <el-button type="success"
                     @click="downFun">下载已上传数据</el-button>

        </el-form-item>

        <el-form-item>
          <el-button type="success"
                     @click="onAddProblemSubmit"
                     style="float:right;">修改题目</el-button>
        </el-form-item>
      </el-form>

    </el-dialog>
  </el-row>
</template>

<script>
export default {
  name: "adminchangepro",
  data () {
    return {

      serviceaddress: "",
      changeproblid: "",
      currentpage: 1,
      gridData: [],
      totalproblem: 0,
      uploadaddress: "/uploadfile/",
      fileList: [],
      loading: false,
      dialogTableVisible: false,
      problemform: {
        problem: "",
        author: sessionStorage.name,
        title: "题目标题",
        des: "题目说明",
        input: "输入说明",
        output: "输出说明",
        sinput: "1 1|#)2 2",
        soutput: "2|#)4",
        source: "GenchOJ",
        time: 1000,
        memory: 64,
        hint: "提示",
        auth: 2,
        tag: "简单题|模拟|贪心",
        level: 3,
        score: 100,
        oj: "GenchOJ",
        template: "*****C++*****\n\n*****C*****\n\n*****Python2*****\n\n*****Python3*****\n\n*****Java*****\n\n*****Swift5.1*****\n\n",
      },
      problemdataform: {
        problem: "",
        title: "题目标题",
        tag: "简单题|模拟|贪心",
        level: 3,
        score: 100
      },
      searchpro: ""
    };
  },
  methods: {
    downFun () {

      var fileurl = this.serviceaddress + "/downloadfile/?name=" + this.changeproblid;
      console.log(fileurl);
      window.open(fileurl);
    },
    problemclick (row) {
      window.open("/problemdetail?problemID=" + row.problem);
    },
    searchtitle () {
      this.currentpage = 1;
      this.$axios
        .get(
          "/problemdata/?limit=20&offset=" +
          (this.currentpage - 1) * 20 +
          "&search=" +
          this.searchpro
        )
        .then(response => {
          this.totalproblem = response.data.count;
          this.gridData = response.data.results;
        });
    },
    handleCurrentChange (val) {
      this.currentpage = val;
      this.$axios
        .get(
          "/problemdata/?limit=20&offset=" +
          (this.currentpage - 1) * 20 +
          "&search=" +
          this.searchpro
        )
        .then(response => {
          this.totalproblem = response.data.count;
          this.gridData = response.data.results;
        });
    },
    handleEdit (row) {
      this.changeproblid = row.problem;
      this.problemform.problem = row.problem;
      this.problemchange(row.problem);
      this.dialogTableVisible = true;
    },
    myupload (f) {
      let param = new FormData(); //创建form对象
      var tail = f.file.name.split(".");
      if (tail[1] == "zip") {
        var newfile = new File([f.file], this.problemform.problem + ".zip");
      }
      else if (tail[1] == "jpg") {
        var newfile = new File([f.file], this.problemform.problem + ".jpg");
      }
      param.append("file", newfile); //通过append向form对象添加数据
      let config = {
        headers: { "Content-Type": "multipart/form-data" }
      }; //添加请求头
      this.$axios
        .post(f.action, param, config) //上传图片
        .then(response => {
          console.log(response.data);
          f.onSuccess(response.data);
        })
        .catch(err => {
          console.log(err);
          f.onError(err);
        });
    },

    handleDel (row) {
      this.$message.error("请把题目权限设置为（私密）即可！");
    },
    problemchange (num) {
      this.$axios
        .get("/problem/" + this.problemform.problem + "/")
        .then(response => {
          this.$axios
            .get("/problemdata/" + this.problemform.problem + "/")
            .then(response2 => {
              response.data.level = response2.data.level;
              response.data.tag = response2.data.tag;
              response.data.score = response2.data.score;
              this.problemform = response.data;
            })
            .catch(error => {
              this.$message.error(
                "服务器错误！" + JSON.stringify(error.response.data)
              );
              this.problemform = {};
            });
        })
        .catch(error => {
          this.$message.error(
            "服务器错误！" + JSON.stringify(error.response.data)
          );
          this.problemform = {};
        });
    },
    handleRemove (file, fileList) {
      this.fileList = [];
    },
    handleExceed (file, fileList) {
      this.$message.error("一次至多只能上传一个文件（ZIP数据文件与图片文件分开上传）！");
    },
    handleChange (file, fileList) {
      var name = file.name;
      var li = name.split(".");
      console.log(li[0]);
      console.log(li[1]);
      this.fileList = fileList;
      if (li[1] != "zip" && li[1] != "jpeg" && li[1] != "jpg" && li[1] != "png") {
        this.$message.error("数据文件名名不正确！后缀应为zip/jpeg/jpg/png");
        this.fileList = [];
      }
    },
    handleError (response, file, fileList) {
      this.$message.error("数据上传失败！" + response);
    },
    async handleSuccess(response, file, fileList) {

      try{
        var response = await this.$axios.put(
          "/problem/" + this.problemform.problem + "/",
          this.problemform
        );
      }
      catch(error){
        console.log(error)
      }
      

      this.problemdataform.problem = this.problemform.problem;
      this.problemdataform.title = this.problemform.title;
      this.problemdataform.level = this.problemform.level;
      this.problemdataform.tag = this.problemform.tag;
      this.problemdataform.score = this.problemform.score;
      this.problemdataform.auth = this.problemform.auth;
      var tag = this.problemdataform.tag.split("|");
      for (var i = 0; i < tag.length; i++) {
        try{
          await this.$axios.post("/problemtag/", {
            tagname: tag[i],
            count: 1
          });
        }catch(error){
          console.log(error)
        }
      }

      try{
        var response2 = await this.$axios.put(
        "/problemdata/" + this.problemform.problem + "/",
        this.problemdataform
        );

        this.$message({
          message: "修改成功！修改题目编号为：" + response2.data.problem,
          type: "success"
        });
        this.fileList = [];
        this.loading = false;
        this.dialogTableVisible = false;
      }
      catch(error){
        console.log(error)
      }


    },

    onAddProblemSubmit () {
      if (this.fileList.length <= 0) {
        this.$confirm(
          "确定修改吗？本次修改没有更新数据",
          "修改题目：" + this.problemform.problem,
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }
        ).then(() => {
          this.handleSuccess(1, 2, 3);
        });

        return;
      }
      this.$confirm(
        "确定修改吗？本次修改将会用新数据覆盖旧数据",
        "修改题目：" + this.problemform.problem,
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.loading = true;
        this.$refs.upload.submit();
      });
    }
  },
  created () {
    this.$axios
      .get(
        "/problemdata/?limit=20&offset=" +
        (this.currentpage - 1) * 20 +
        "&search=" +
        this.searchpro
      )
      .then(response => {
        this.totalproblem = response.data.count;
        this.gridData = response.data.results;
      });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
