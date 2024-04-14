<template>
  <div>

    <!--条件搜索区域-->
    <el-row>
      <el-col :span="24">
        <el-card header="读者列表">
          <el-form :inline="true">
            <el-form-item label="学号">
              <el-input v-model="searchStudentID" placeholder="请输入学号" clearable>
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-button icon="el-icon-search" @click="getStudentList">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>


    <!--数据显示区域-->
    <el-row>
      <el-col :span="24">
        <el-card>
          <el-header style="background-color: #ffffff">
            <el-button @click="handleSave()" icon="el-icon-circle-plus-outline">添加学生信息</el-button>
          </el-header>
          <el-table :data="tableData" style="width: 100%">
            <el-table-column width="230" prop="studentID" label="学号">
            </el-table-column>

            <el-table-column width="260" prop="date" label="人脸图片">
              <template slot-scope="scope">
                <el-button icon="el-icon-picture-outline" size="mini"
                  @click="selectImg(scope.row.faceBase)">查看人脸</el-button>
              </template>
            </el-table-column>

            <el-table-column prop="nickname" label="昵称" width="230">
            </el-table-column>

            <el-table-column prop="creditScore" label="信誉分" width="230">
            </el-table-column>

            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button icon="el-icon-edit" size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                <el-popconfirm :title="('确认删除学号：' + scope.row.studentID + '的学生数据吗?')"
                  @confirm="handleDelete(scope.$index, scope.row)">
                  <el-button style="margin-left: 20px" icon="el-icon-delete" size="mini" type="danger"
                    slot="reference">删除</el-button>
                </el-popconfirm>

              </template>
            </el-table-column>

          </el-table>
        </el-card>
      </el-col>
    </el-row>

    <!--分页-->
    <el-row class="page-row">
      <el-pagination class="el-pagination" @size-change="handleSizeChange" @current-change="handleCurrentChange"
        layout="total, sizes, prev, pager, next, jumper" :page-sizes="[5, 10, 30, 50]" :current-page="current"
        :page-size="size" :total="total">
      </el-pagination>
    </el-row>



    <!--查看人脸图片-->
    <el-dialog title="人脸图片" :visible.sync="selectImgDialog" width="30%">
      <img style="width: 100%;" :src="currentImg">
    </el-dialog>


    <!-- 添加/修改-->
    <el-dialog :title="saveAndUpdateTitle" :visible.sync="saveAndUpdateDialog" :before-close="handleClose" width="600px">
      <el-form :model="studentObj" :rules="rules" ref="ruleForm" label-width="80px">

        <el-form-item label="学号" prop="studentID">
          <el-input placeholder="请输入学号" v-model="studentObj.studentID" :disabled="!this.addAndUpdateState"></el-input>
        </el-form-item>

        <el-form-item label="密码" prop="password">
          <el-input placeholder="请输入密码" v-model="studentObj.password"></el-input>
        </el-form-item>

        <el-form-item label="信誉分">
          <el-input-number v-model="studentObj.creditScore" controls-position="right" :min="1"
            :max="100"></el-input-number>
        </el-form-item>

        <el-form-item label="昵称" prop="nickname" style="margin-bottom: 40px;">
          <el-input placeholder="请输入昵称" v-model="studentObj.nickname"></el-input>
        </el-form-item>

        <el-form-item label="人脸数据" class="login-mid-mid" prop="faceBase">
          <!--捕获人脸区域-->
          <div class="savevideoCamera-canvasCamera">
            <video id="videoCamera2" :width="videoWidth" :height="videoHeight" autoplay></video>
            <canvas style="display: none" id="canvasCamera2" :width="videoWidth" :height="videoHeight"></canvas>

            <!--人脸特效区域-->
            <div v-if="faceImgState" class="saveface-special-effects-2"></div>
            <div v-else class="saveface-special-effects"></div>
            <el-input type="hidden" v-model="studentObj.faceBase"></el-input>
          </div>

          <!--拍照显示区域-->
          <div class="savefaceBase" id="faceBase">

          </div>

          <!--按钮区域-->
          <div class="saveface-btn" v-if="cameraState">
            <el-button @click="enteringFace">{{ studentObj.faceBase === '' ? '录入人脸' : '重新录入' }}</el-button>
          </div>
          <div class="saveface-btn" v-else>
            <el-button>摄像头正在加载...</el-button>
          </div>

        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">{{ this.addAndUpdateState ? '立即创建' :
            '保存修改' }}</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
        </el-form-item>


      </el-form>

    </el-dialog>

  </div>
</template>

<script>
import $camera from "../../camera/index.js";
export default {
  data() {
    return {
      size: 5,
      current: 1,
      total: 0,
      videoWidth: 200,
      videoHeight: 200,
      faceImgState: false,
      searchStudentID: '',
      tableData: [],
      selectImgDialog: false,
      currentImg: '',
      saveAndUpdateDialog: false,
      addAndUpdateState: false,
      saveAndUpdateTitle: '',
      studentObj: {
        studentID: null,
        faceBase: '',
        password: '',
        nickname: '',
        creditScore: 100
      },
      rules: {
        studentID: [
          { required: true, message: '学号不能为空', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' }
        ],
        nickname: [
          { required: true, message: '昵称不能为空', trigger: 'blur' }
        ],
        faceBase: [
          { required: true, message: '人脸数据不能为空' }
        ]
      },
      faceOption: {},
      cameraState: false
    }
  },
  watch: {
    saveAndUpdateDialog: {
      handler(newValue, oldValue) {
        // 监听弹出层 设置摄像头是否开启
        if (newValue) {
          // 延迟加载 不然document.getElementById找不到
          setTimeout(() => {
            // 开启摄像头
            this.faceOption = $camera.getCamera({
              videoWidth: 200,
              videoHeight: 200,
              thisCancas: null,
              thisContext: null,
              thisVideo: null,
              canvasId: 'canvasCamera2',
              videoId: 'videoCamera2'
            });
          }, 1000)
        } else {
          let faceBase = document.getElementById("faceBase");
          faceBase.innerHTML = ""
          if (this.faceOption.thisVideo != null) {
            // 不等于null 关闭摄像头
            this.faceOption.thisVideo.srcObject.getTracks()[0].stop();
          }
        }
      }
    }
  },
  // 获取学生数据
  created() {
    this.getStudentList()
  },
  methods: {
    // 录入
    enteringFace() {
      this.studentObj.faceBase = $camera.draw(this.faceOption)
      let faceBase = document.getElementById("faceBase");
      faceBase.innerHTML = "<img src='" + this.studentObj.faceBase + "' />"
    },
    // 增加|修改
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.$http.post("/student/" + (this.addAndUpdateState ? 'register' : 'update'), this.studentObj).then(res => {
            this.$message({
              showClose: true,
              message: res.data.msg,
              type: 'success',
              duration: 3000
            })
            this.getStudentList()
            this.saveAndUpdateDialog = false
          })
        } else {
          return false;
        }
      });
    },
    // 重置表单
    resetForm(formName) {
      let faceBase = document.getElementById("faceBase");
      faceBase.innerHTML = ""
      this.studentObj = {
        studentID: null,
        faceBase: '',
        password: '',
        nickname: '',
        creditScore: 100
      }
    },
    // 查看人脸
    selectImg(faceBase) {
      this.selectImgDialog = true
      this.currentImg = faceBase
    },
    // 获取人脸列表
    getStudentList() {
      this.$http.get('/student/list', {
        params: {
          current: this.current,
          size: this.size,
          studentID: this.searchStudentID
        }
      }).then(res => {
        this.tableData = res.data.data.records
        this.size = res.data.data.size
        this.current = res.data.data.current
        this.total = res.data.data.total
      })
    },
    // 分页
    handleSizeChange(val) {
      this.size = val
      this.getStudentList()
    },
    // 分页
    handleCurrentChange(val) {
      this.current = val
      this.getStudentList()
    },
    // 关闭弹出层
    handleClose() {
      this.saveAndUpdateDialog = false
      this.cameraState = false
      this.resetForm('ruleForm')
    },
    // 设置按钮的延迟加载
    setCameraState() {
      setTimeout(() => {
        this.cameraState = true
      }, 1500)
    },
    // 添加
    handleSave() {
      this.setCameraState()
      this.addAndUpdateState = true
      this.saveAndUpdateDialog = true
      this.saveAndUpdateTitle = '添加学生信息'
    },
    // 修改
    handleEdit(index, row) {
      this.setCameraState()
      this.addAndUpdateState = false
      this.saveAndUpdateTitle = '修改学生信息'
      this.saveAndUpdateDialog = true
      this.studentObj = { ...row };
      this.$nextTick(() => {
        let faceBase = document.getElementById("faceBase");
        if (faceBase) {
          faceBase.innerHTML = "<img src='" + this.studentObj.faceBase + "' />"
        }
      });
    },
    //删除
    handleDelete(index, row) {
      this.$http.delete("/student/" + row.studentID).then(res => {
        this.getStudentList();
        this.$message.success(res.data.msg)
      })
    }
  }
}
</script>

<style>
@import "./index.css";
</style>