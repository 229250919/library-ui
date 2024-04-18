<template>
  <div>


    <!--条件搜索区域-->
    <el-row>
      <el-col :span="24">
        <el-card header="签到列表">
          <el-form :inline="true">
            <el-form-item label="学号">
              <el-input v-model="searchStudentID" placeholder="请输入学号" clearable>
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-button icon="el-icon-search" @click="getCheckinList">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>

    <!--数据表格-->
    <el-row>
      <el-col :span="24">
        <el-card>
          <el-table :data="tableLogData" style="width: 100%">
            <el-table-column width="200" prop="checkinID" label="签到号">
            </el-table-column>


            <el-table-column width="200" prop="studentID" label="学号">
            </el-table-column>


            <el-table-column width="200" prop="enterTime" label="进入时间">
            </el-table-column>

            <el-table-column prop="leaveTime" label="离开时间">
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

  </div>
</template>

<script>
export default {
  name: "index",
  data() {
    return {
      size: 5,
      current: 1,
      total: 0,
      searchStudentID: '',
      tableLogData: []
    }
  },
  // 获取人脸数据
  created() {
    this.getCheckinList()
  },
  methods: {
    // 分页
    handleSizeChange(val) {
      this.size = val
      this.getCheckinList()
    },
    // 分页
    handleCurrentChange(val) {
      this.current = val
      this.getCheckinList()
    },
    // 日志列表
    getCheckinList() {
      this.$http.get('/checkinRecord/list', {
        params: {
          current: this.current,
          size: this.size,
          studentID: this.searchStudentID
        }
      }).then(res => {
        this.tableLogData = res.data.data.records
        this.size = res.data.data.size
        this.current = res.data.data.current
        this.total = res.data.data.total
      })
    },
  }
}
</script>

<style >
.page-row {
  margin-top: 20px;
  text-align: center;
}
</style>