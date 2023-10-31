<template>
  <div class="login" v-if="login_box">
    用户名
    <el-input v-model="form.username" placeholder="请输入内容" style="width: 180px"></el-input>
    密 码
    <el-input placeholder="请输入密码" v-model="form.password" style="width: 180px" show-password></el-input>
    <el-button type="primary" @click="login">登 录</el-button>
  </div>
  <div class="table_box" v-show="!login_box">
    <template v-if="!login_box">
      姓名：
      <el-input v-model="query.name" placeholder="请输入内容" style="width: 150px"></el-input>
      生日：
      <el-date-picker
          v-model="query.startTime"
          type="date"
          placeholder="选择日期">
      </el-date-picker>
      -到-
      <el-date-picker
          v-model="query.endTime"
          type="date"
          placeholder="选择日期">
      </el-date-picker>
      <el-button type="primary" @click="getStudent">搜索</el-button>
      <el-button type="primary" @click="addStudentPage">添加学生</el-button>
      <el-table
          :data="tableData.list"
          style="width: 100%">
        <el-table-column
            prop="id"
            label="编号"
            width="100">
        </el-table-column>
        <el-table-column
            prop="name"
            label="姓名"
            width="100">
        </el-table-column>
        <el-table-column
            prop="age"
            label="年龄">
        </el-table-column>
        <el-table-column
            prop="sex"
            label="性别">
        </el-table-column>
        <el-table-column
            prop="address"
            label="家庭住址">
        </el-table-column>
        <el-table-column
            prop="birthday"
            label="生日">
<!--          <template #default="scope">-->
<!--            {{scope.row.birthday|format('yyyy-MM-dd')}}-->
<!--          </template>-->
        </el-table-column>

        <el-table-column
            prop="img"
            label="头像">
          <template #default="scope">
            <el-avatar
                :src="scope.row.img"
            />
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template #default="scope">
            <el-button
                size="small"
                @click="handleEdit( scope.row)">编辑
            </el-button>
            <el-button
                size="small"
                type="danger"
                @click="handleDelete( scope.row.id)">删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--   页码   -->
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="tableData.pageNum"
          :page-sizes="[5, 10, 15, 20]"
          :page-size="5"
          layout="total, sizes, prev, pager, next, jumper"
          :total="tableData.total">
      </el-pagination>
    </template>
  </div>
  <el-dialog
      :title="model.title"
      v-model="dialogFormVisible"
      width="30%">
    <el-form :model="student_form">
      <el-form-item label="姓名" :label-width="formLabelWidth">
        <el-input v-model="student_form.name" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="年龄" :label-width="formLabelWidth">
        <el-input v-model="student_form.age" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="性别" :label-width="formLabelWidth">
        <el-radio-group v-model="student_form.sex">
          <el-radio label="男">男</el-radio>
          <el-radio label="女">女</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="生日" :label-width="formLabelWidth">
        <el-date-picker
            v-model="student_form.birthday"
            type="date"
            placeholder="选择日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="头像">
        <el-upload
            class="avatar-uploader"
            action="/images/upload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
          <img v-if="imageUrl" :src="imageUrl" class="avatar" alt=""/>
          <el-icon v-else class="avatar-uploader-icon">
            <Plus/>
          </el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item label="家庭住址" :label-width="formLabelWidth">
        <el-input v-model="student_form.address" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisible = false">取 消</el-button>
      <el-button type="primary" @click="addStudentFun">确 定</el-button>
    </div>
  </el-dialog>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      model: {
        title: '',
        url: '',
      },
      student_form: {
        name: '',
        age: '',
        sex: '',
        birthday: '',
        img: '',
      },
      dialogFormVisible: false,
      form: {
        username: '',
        password: ''
      },
      tableData: {},
      page: {
        pageNum: 1,
        pageSize: 5
      },
      query: {
        name: '',
        startTime: '',
        endTime: '',
      },
      login_box: true,
      imageUrl: '',
    }
  },
  methods: {
    login() {
      axios.post(`/user/login`, this.form).then(res => {
        if (res.data != null && res.data !== '') {
          this.$message.success('登陆成功')
          this.getStudent()
          this.login_box = false
        } else {
          this.$message.error('登陆失败')
        }
        console.log(res)
      })
    },
    getStudent() {
      axios.post(`/student/getStudentAll?pageNum=${this.page.pageNum}&pageSize=${this.page.pageSize}`, this.query).then(res => {
        this.tableData = res.data
      })
    },
    handleCurrentChange(val) {
      this.page.pageNum = val
      this.getStudent()
    },
    handleSizeChange(val) {
      this.page.pageSize = val
      this.getStudent()
    },
    addStudentPage() {
      this.student_form = {};
      this.student_form.img = ''
      this.dialogFormVisible = true
      this.model.title = '添加学生'
      this.model.url = '/student/addStudent'
      console.log(this.model)
      console.log(this.dialogFormVisible)
    },
    addStudentFun() {
      axios.post(this.model.url, this.student_form).then(res => {
        if (res.data) {
          this.$message.success('操作成功');
          this.student_form = {};
          this.dialogFormVisible = false
          this.getStudent();
        } else {
          this.$message.error('操作失败');
        }
      })
    },
    handleDelete(val) {
      //提示是否删除
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        axios.delete(`/student/deleteStudent?id=${val}`).then(res => {
          if (res.data) {
            this.$message.success('删除成功');
            this.getStudent()
          } else {
            this.$message.error('删除失败');
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    //编辑
    handleEdit(val) {
      this.student_form = val;
      this.student_form.img = ''
      this.dialogFormVisible = true
      this.model.title = '修改学生信息'
      this.model.url = '/student/updateStudent'
      console.log(this.model)
      console.log(this.dialogFormVisible)
    },

    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
      this.student_form.img = res
    },
    beforeAvatarUpload(file) {
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (isLt2M) {

        return true;
      }
      this.$message.error('上传头像图片大小不能超过 2MB!');
      return false;
    }
  },

  filters:{
    format(value,arg){
      function dateFormat(date, format) {
        if (typeof date === "string") {
          const mts = date.match(/(\/Date\((\d+)\)\/)/);
          if (mts && mts.length >= 3) {
            date = parseInt(mts[2]);
          }
        }
        date = new Date(date);
        if (!date || date.toUTCString() === "Invalid Date") {
          return "";
        }
        const map = {
          "M": date.getMonth() + 1, //月份
          "d": date.getDate(), //日
          "h": date.getHours(), //小时
          "m": date.getMinutes(), //分
          "s": date.getSeconds(), //秒
          "q": Math.floor((date.getMonth() + 3) / 3), //季度
          "S": date.getMilliseconds() //毫秒
        };
        format = format.replace(/([yMdhmsqS])+/g, function (all, t) {
          let v = map[t];
          if (v !== undefined) {
            if (all.length > 1) {
              v = '0' + v;
              v = v.substr(v.length - 2);
            }
            return v;
          } else if (t === 'y') {
            return (date.getFullYear() + '').substr(4 - all.length);
          }
          return all;
        });
        return format;
      }

      return dateFormat(value,arg);

    }
  },
}
</script>


<style>

.avatar-uploader .avatar {
  width: 178px;
  height: 178px;
  display: block;
}

.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}

.demo-type {
  display: flex;
}

.demo-type > div {
  flex: 1;
  text-align: center;
}

.demo-type > div:not(:last-child) {
  border-right: 1px solid var(--el-border-color);
}
</style>
