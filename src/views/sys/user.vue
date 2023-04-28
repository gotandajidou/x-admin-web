<template>
  <div>
    <el-card id="search">
      <el-row>
        <el-col :span="20">
          <el-input v-model="searchModel.username" placeholder="ユーザー名" />
          <el-input v-model="searchModel.phone" placeholder="電話番号" />
          <el-button type="primary" @click="getUserList" round icon="el-icon-search">検索</el-button>
        </el-col>
        <el-col :span="4" align="right">
          <el-button @click="openEditUI(null)" type="primary" round icon="el-icon-plus" />
        </el-col>
      </el-row>
    </el-card>
    <el-card>
      <el-table :data="useList" stripe style="width: 100%">
        <el-table-column type="index" label="#" width="80" />
        <el-table-column prop="id" label="ユーザID" width="180" />
        <el-table-column prop="username" label="ユーザー名" width="180" />
        <el-table-column prop="phone" label="電話番号" width="180" />
        <el-table-column prop="status" label="状態" width="180" >
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1">正常</el-tag>
            <el-tag v-else type="danger">禁止</el-tag>
          </template>
        </el-table-column>
        
        <el-table-column prop="email" label="メールアドレス" />
        <el-table-column label="操作" width="180" >
          <template slot-scope="scope">
            <el-button @click="openEditUI(scope.row.id)" type="primary" icon="el-icon-edit" size="mini" circle></el-button>
            <el-button @click="deleteUser(scope.row)" type="danger" icon="el-icon-delete" size="mini" circle></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分页组件 -->
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
      :current-page="searchModel.pageNo" :page-sizes="[5, 10, 20, 50]" :page-size="searchModel.pageSize"
      layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
    <!-- 用户信息编辑对话框 -->
    <el-dialog @close="clearForm" :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="userForm" ref="userFormRef" :rules="rules">
        <el-form-item label="ユーザー名" prop="username" :label-width="formLabelWidth">
          <el-input v-model="userForm.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item v-if="userForm.id == null || userForm.id == undefined"
          label="パスワード" 
          prop="password" 
          :label-width="formLabelWidth"
          >
          <el-input type="password" v-model="userForm.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="電話番号" :label-width="formLabelWidth">
          <el-input v-model="userForm.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="アカウント状態" :label-width="formLabelWidth">
          <el-switch 
            v-model="userForm.status"
            :active-value="1"
            :inactive-value="0" 
            >
          </el-switch>
        </el-form-item>
        <el-form-item label="メールアドレス" prop="email" :label-width="formLabelWidth">
          <el-input v-model="userForm.email" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveUser">確 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from '@/api/userManage'
export default {
  deleteUser(user){
      this.$confirm(`ユーザー ${user.username}を削除してもいいでしょうか?`, '注意', {
          confirmButtonText: '確定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          userApi.deleteUserById(user.id).then(response => {
            this.$message({
              type: 'success',
              message: response.message
            });
            this.getUserList();
          });
          
        }).catch(() => {
          this.$message({
            type: 'info',
            message: 'ユーザーの削除をキャンセルします。'
          });          
        });
    },
  data() {
    var checkEmail = (rule, value, callback) => {
      var reg =
        /^[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*@[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*\.[a-z]{2,}$/;
        if (!reg.test(value)) {
          return callback(new Error('メールアドレスの形式が間違っています。'));
        }
        callback();
        
      };
    return {
      formLabelWidth: '130px',
      userForm: {},
      dialogFormVisible: false,
      title: "",
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 10
      },
      useList: [],
      rules:{
        username: [
            { required: true, message: 'ユーザー名を入力してください。', trigger: 'blur' },
            { min: 3, max: 50, message: '長さが3から50文字の間である必要があります。', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '初期パスワードを入力してください。', trigger: 'blur' },
            { min: 6, max: 16, message: '長さが6から16文字の間である必要があります。', trigger: 'blur' }
          ],
          email: [
            { required: true, message: 'メールアドレスを入力してください。', trigger: 'blur' }, 
            {validator:checkEmail,trigger:'blur'}          
          ],
      }
    }
  },
  methods: {
    saveUser() {
      // 触发表单验证
      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          // 提交请求给后台
          userApi.saveUser(this.userForm).then(response => {
            // 成功提示
            this.$message({
              message: response.message,
              type: 'success'
            });
            // 关闭对话框
            this.dialogFormVisible = false;
            // 刷新表格
            this.getUserList();
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
      
    },
    clearForm(){
      this.userForm={}
      this.$refs.userFormRef.clearValidate();
    },
    openEditUI(id) {
      if( id == null){
        this.title = "新規ユーザー追加";
      }else{
        this.title = "ユーザーの変更";
        // 根据id查询用户数据
        userApi.getUserById(id).then(response => {
          this.userForm = response.data;
        });
      }
      this.dialogFormVisible = true;
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize
      this.getUserList()
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo
      this.getUserList()
    },
    getUserList() {
      userApi.getUserList(this.searchModel).then(response => {
        this.useList = response.data.rows
        this.total = response.data.total
      })
    }
  },
  created() {
    this.getUserList()
  }
}
</script>

<style>
#search .el-input {
  width: 200px;
  margin-right: 10px;
}

.el-dialog.el-input {
  width: 85%;
}
</style>
