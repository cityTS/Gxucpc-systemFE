<template>
  <header class="header-user">用户管理</header>
  <div class="add-user">
    <el-button type="success" @click="showAddUserDialog()" plain>添加用户</el-button>
  </div>
  <el-table :data="tableData" style="width: 100%">
    <el-table-column label="Username" width="180" prop="username"/>
    <el-table-column label="Create Time" width="180" prop="createTime"/>
    <el-table-column label="Last Login" width="180" prop="lastLogin"/>
    <el-table-column label="Real Name" width="180" prop="realName"/>
    <el-table-column label="Email" width="180" prop="email"/>
    <el-table-column label="User Type" width="180" prop="userType"/>

    <el-table-column label="Operations">
      <template #default="scope">
        <el-button size="small" @click="handleEdit(scope.$index, scope.row)">Edit</el-button>
        <el-button size="small" type="danger" @click="handleDelete(scope.$index, scope.row)">Delete</el-button>
      </template>
    </el-table-column>
  </el-table>
  <!-- 分页 -->
  <el-pagination background layout="total, prev, pager, next" v-model:currentPage="pagingComponent.currentPage"
                 :total="pagingComponent.total" @current-change="currentChange"/>
  <!-- 编辑 -->
  <el-dialog v-model="dialogTableVisible" title="用户信息">
    <el-form ref="user-edit-form" :model="editData" :rules="rulesEdit" label-width="120px" label-position="left">
      <el-row :gutter="20">
        <el-col :span="12">
          <el-form disabled="true" label-width="120px" label-position="left">
            <el-form-item label="Username" prop="username">
              <el-input v-model="editData.username"></el-input>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="12">
          <el-form-item label="Real Name" prop="realName">
            <el-input v-model="editData.realName"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="Email" prop="email">
            <el-input v-model="editData.email"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="Password" prop="password">
            <el-input v-model="editData.password"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="User Type" prop="userType">
            <el-select v-model="editData.userType">
              <el-option label="Admin" value="Admin"></el-option>
              <el-option label="Super Admin" value="Super Admin"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <span class="dialog-footer">
      <el-button @click="showUserDialog()">Cancel</el-button>
      <el-button type="primary" @click="saveUser()">Confirm</el-button>
    </span>
  </el-dialog>
  <!-- 添加 -->
  <el-dialog v-model="dialogAddTableVisible" title="添加用户">
    <el-form ref="user-add-form" :model="addData" :rules="rules" label-width="120px" label-position="left">
      <el-row :gutter="20">
        <el-col :span="12">
          <el-form-item label="Username" prop="username">
            <el-input v-model="addData.username"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="Real Name" prop="realName">
            <el-input v-model="addData.realName"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="Email" prop="email">
            <el-input v-model="addData.email"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="Password" prop="password">
            <el-input v-model="addData.password"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="User Type" prop="userType">
            <el-select v-model="addData.userType">
              <el-option label="Admin" value="Admin"></el-option>
              <el-option label="Super Admin" value="Super Admin"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <span class="dialog-footer">
      <el-button @click="showAddUserDialog()">Cancel</el-button>
      <el-button type="primary" @click="addUser()">Confirm</el-button>
    </span>
  </el-dialog>
</template>

<script>
import {getFormtTime} from "@/assets/js/DateUtils.js";
import {ElLoading} from "element-plus";

export default {
  data() {
    return {
      dialogTableVisible: false,
      dialogAddTableVisible: false,
      pagingComponent: {
        total: 0,
        currentPage: 1,
        numberPerPage: 10,
      },
      rules: {
        username: [{required: true, validator: "必填项", trigger: "blur"}],
        realName: [{required: true, validator: "必填项", trigger: "blur"}],
        email: [{required: true, validator: "必填项", trigger: "blur"}],
        userType: [{required: true, validator: "必填项", trigger: "blur"}],
        password: [{required: true, validator: "必填项", trigger: "blur"}],
      },
      rulesEdit: {
        username: [{required: true, validator: "必填项", trigger: "blur"}],
        realName: [{required: true, validator: "必填项", trigger: "blur"}],
        email: [{required: true, validator: "必填项", trigger: "blur"}],
        userType: [{required: true, validator: "必填项", trigger: "blur"}],
      },
      tableData: [
        // {
        //   id: "undefined",
        //   username: "undefined",
        //   createTime: "undefined",
        //   lastLogin: "undefined",
        //   realName: "undefined",
        //   email: "undefined",
        //   userType: "undefined",
        //   password: "undefined",
        // },
      ],
      editData: {
        username: undefined,
        realName: undefined,
        email: undefined,
        userType: undefined,
        password: null,
      },
      addData: {
        username: undefined,
        realName: undefined,
        email: undefined,
        userType: undefined,
        password: null,
      },
    };
  },
  mounted() {
    this.getUserInfo();
  },
  methods: {
    // 换页
    currentChange(number) {
      this.pagingComponent.currentPage = number;
      this.getUserInfo();
    },
    jsonClone(obj) {
      // js深复制
      return JSON.parse(JSON.stringify(obj));
    },
    handleEdit(index, row) {
      this.showUserDialog();
      this.editData = this.jsonClone(row);
      this.editData.password = "";
    },
    async handleDelete(index, row) {
      let loading = ElLoading.service({
        lock: true,
        text: '删除中，请稍后...',
        background: 'rgba(0, 0, 0, 0.7)',
      })
      await this.$http
          .put("/admin/user/delete" + "?id=" + row.id + "&name=" + row.username)
          .then((res) => {
            if (res.statusCode === 50000) {
              this.$message.success("删除成功");
            } else {
              this.$message.error(res.message);
            }
            this.getUserInfo();
          })
          .catch(() => {
            this.$message.error("网络故障或系统故障");
          });
      loading.close();

    },
    showUserDialog() {
      this.dialogTableVisible = !this.dialogTableVisible;
    },
    showAddUserDialog() {
      this.dialogAddTableVisible = !this.dialogAddTableVisible;
      for (var index in this.addData) {
        this.addData[index] = undefined;
      }
    },
    async saveUser() {
      let vis = this.$refs['user-edit-form'].validate();
      if (!vis) return;
      let loading = ElLoading.service({
        lock: true,
        text: '保存中，请稍后...',
        background: 'rgba(0, 0, 0, 0.7)',
      })
      var isChangePasswordItem = 1;
      if (this.editData.password !== "") {
        isChangePasswordItem = 2;
      }
      this.editData.createTime = null
      this.editData.lastLogin = null
      await this.$http
          .put("/admin/user", this.editData, {headers: {isChangePassword: isChangePasswordItem}})
          .then((res) => {
            if (res.statusCode === 50000) {
              this.$message.success("修改成功");
              this.showUserDialog();
              this.getUserInfo();
            } else {
              this.$message.error(res.message);
            }
          })
          .catch(() => {
            this.$message.error("网络故障或系统故障");
          });
      loading.close();
    },
    async addUser() {
      let vis = this.$refs['user-add-form'].validate();
      if (!vis) return;
      let loading = ElLoading.service({
        lock: true,
        text: '添加中，请稍后...',
        background: 'rgba(0, 0, 0, 0.7)',
      })
      await this.$http
          .post("/admin/user", this.addData)
          .then((res) => {
            if (res.statusCode === 50000) {
              this.$message.success("添加成功");
              this.showAddUserDialog();
              this.getUserInfo();
            } else {
              this.$message.error(res.message);
            }
          })
          .catch(() => {
            this.$message.error("网络故障或系统故障");
          });
      this.addData = {};
      loading.close();
    },
    getUserInfo() {
      var ask =
          "?currentPage=" +
          this.pagingComponent.currentPage +
          "&numberPerPage=" +
          this.pagingComponent.numberPerPage;
      this.$http.get("/admin/user" + ask).then((res) => {
        if (res.statusCode === 50000) {
          this.pagingComponent.total = res.data.total;
          this.tableData = res.data.tableData;
          // 格式化时间
          for (var index in this.tableData) {
            this.tableData[index].createTime = getFormtTime(this.tableData[index].createTime, true)
            if (this.tableData[index].lastLogin === 0) this.tableData[index].lastLogin = "Not Logined"
            else this.tableData[index].lastLogin = getFormtTime(this.tableData[index].lastLogin, true)
            // 格式化权限
            this.tableData[index].userType = this.tableData[index].userType == 1 ? "Super Admin" : "Admin";
          }
        } else {
          this.$message.error(res.message);
        }
      });
    },
  },
};
</script>

<style>
.header-user {
  background-color: #ffffff;
  padding: 10px;
}

.add-user {
  width: 100%;
  text-align: right;
  margin-right: 5%;
  background-color: #ffffff;
}
</style>
