<template>
  <div>
<!--搜索栏-->
    <el-card id="search">
      <el-input v-model="searchModel.username" placeholder="用户名" clearable></el-input>
      <el-input v-model="searchModel.phone" placeholder="电话" clearable></el-input>
      <el-button @click="openEditUI(null)" type="primary" icon="el-icon-plus" circle></el-button>
      <el-button @click="getUserList" type="primary" round icon="el-icon-search"> 查询</el-button>
    </el-card>

<!--结果列表-->
    <el-card id="search">
        <el-table :data="userList" stripe style="width: 100%">
          <el-table-column label="序号" width="80">
            <!--(pageNo-1)*pageSize + index + 1         作用域插槽-->
            <template slot-scope="scope">
              {{(searchModel.pageNo-1) * searchModel.pageSize + scope.$index + 1}}
            </template>
          </el-table-column>
          <el-table-column prop="id" label="用户ID" width="90">
          </el-table-column>
          <el-table-column prop="username" label="用户名" width="150">
          </el-table-column>
          <el-table-column prop="phone" label="电话" width="150">
          </el-table-column>
          <el-table-column prop="email" label="邮箱地址" width="150">
          </el-table-column>
          <el-table-column prop="status" label="用户状态">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.status == 1" >启用</el-tag>   <!--scope可以获取当前行的参数内容-->
              <el-tag v-else type="danger">禁用</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="100">
            <template slot-scope="scope">
              <el-button @click="openEditUI(scope.row.id)" type="primary" icon="el-icon-edit" size="mini" circle></el-button>
              <el-button @click="deleteUser(scope.row)" type="danger" icon="el-icon-delete" size="mini" circle></el-button>
            </template>
          </el-table-column>
        </el-table>
    </el-card>
    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="searchModel.pageNo"
        :page-sizes="[5, 10]"
        :page-size="searchModel.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>

  <!--用户信息编辑对话框-->
  <el-dialog @close="clearForm" :title="title" :visible.sync="dialogFormVisible">
    <el-form :model="userForm" ref="userFormRef" :rules="rules">
      <el-form-item label="用户名:" prop="username" :label-width="formLabelWidth">
        <el-input v-model="userForm.username" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item v-if="userForm.id == null || userForm.id == undefined" label="密码:" prop="password" :label-width="formLabelWidth">
        <el-input v-model="userForm.password" type="password" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="联系电话:" :label-width="formLabelWidth">
        <el-input v-model="userForm.phone" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="电子邮件:" prop="email" :label-width="formLabelWidth">
        <el-input v-model="userForm.email" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="用户状态:" :label-width="formLabelWidth">
        <el-switch
          v-model="userForm.status"
          active-color="#13ce66"
          inactive-color="#ff4949"
          active-value = 1
          inactive-value = 0
          >
        </el-switch>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisible = false">取 消</el-button>
      <el-button type="primary" @click="saveUser" >确 定</el-button>
    </div>
  </el-dialog>
  </div>
</template>

<script>
// 查询对象
import userApi from '@/api/userManage'
export default {
  data() {
    return {
      title: "",
      dialogFormVisible: false,
      userForm:{},
      formLabelWidth: '130px',
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 5
      },
      userList: [],
      rules: {
        username: [
          {required: true, message: '请输入用户名', trigger: 'blur' },
          {min: 1, max: 15, message: '长度为1-15个字符' ,trigger: 'blur'}
        ],
        password: [
          {required: true, message: '请输入密码', trigger: 'blur' },
          {min: 6, max: 30, message: '长度为6-30个字符', trigger: 'blur'}
        ],
        email: [
          {required: true, message: '请输入邮箱', trigger: 'blur' }
        ]
      }
    }
  },
  //`此操作将永久删除用户${user.username}, 是否继续?`
  methods: {
    deleteUser(user){
      this.$confirm('此操作将永久删除用户' + user.username, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        userApi.deleteUserById(user.id).then(response => {
          this.$message({
            type: 'success',
            message: response.message
          });
        });
        this.getUserList();//刷新数据
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });

    },
    saveUser(){
      //触发表单验证
      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          //提交给后台，在saveUser中自带判断更简洁
          userApi.saveUser(this.userForm).then(response => {
            //给予成功提示和关闭对话框
            this.$message({
              message: response.message,
              type: 'success'
            });
            this.dialogFormVisible = false;
            //刷新表格
            this.getUserList();
          })
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    clearForm(){
      this.userForm = {}
      this.$refs.userFormRef.clearValidate();//关闭后清理校验信息
    },
    openEditUI(id) {
      if(id == null){
        this.title = '新增用户';
      } else{
        this.title = '修改用户';
        //根据id查询数据
        userApi.getUserById(id).then(response => {
          this.userForm = response.data;
        });
      }
      this.dialogFormVisible = true
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize // 更新页数
      this.getUserList() // 再做新查询
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo
      this.getUserList()
    },
    getUserList() {
      userApi.getUserList(this.searchModel).then(response => {
        this.userList = response.data.rows
        this.total = response.data.total
      })
    }
  },
  created() { // 钩子函数
    this.getUserList()
  },
}
</script>

<style scoped>
  #search .el-input{
    margin: 7px;
  }
  #search .el-button{
    margin-right: 5px;
    margin-bottom: 10px;
    float: right;
  }
  .el-dialog .el-input{
    width: 85%;
  }
</style>
