<template>
  <div>


    <!--条件搜索区域-->
    <el-row>
      <el-col :span="24">
        <el-card header="座位管理">
          <el-form :inline="true">
            <el-form-item label="楼层">
              <el-select v-model="floor" placeholder="请选择楼层" @change="handleFloorChange">
                <el-option v-for="item in floors" :key="item" :label="item" :value="item">
                </el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <el-form :inline="true">
            <el-form-item label="楼层">
              <el-input placeholder="请输入楼层" v-model="newSeat.floor" clearable>
              </el-input>
            </el-form-item>
            <el-form-item label="座位号">
              <el-input placeholder="请输入座位号" v-model="newSeat.seatNumber" clearable>
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-button icon="el-icon-add-location" @click="addSeat">添加座位</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>

      <!-- 温度、湿度、开关 -->
      <el-col :span="24" v-if="temperature !== null || humidity !== null">
        <el-card header="环境数据和设备控制" style="margin-top: 20px;">
          <el-form :inline="true">
            <el-form-item v-if="temperature !== null">
              <span style="font-size: 30px;">温度：{{ temperature }} °C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
            </el-form-item>
            <el-form-item v-if="humidity !== null">
              <span style="font-size: 30px;">湿度：{{ humidity }} %&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
            </el-form-item>
            <el-form-item v-if="lamp !== null">
              <label style="font-size: 30px;">灯：</label>
              <el-switch v-model="lamp" active-color="#13ce66" inactive-color="#C8C8C8" active-value="on"
                inactive-value="off" active-text="开" inactive-text="关" @change="switchLamp()"></el-switch>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>

    <!--数据表格-->
    <el-row>
      <el-col :span="24">
        <el-card>
          <el-table :data="seats" style="width: 100%">
            <el-table-column width="200" prop="floor" label="楼层">
            </el-table-column>


            <el-table-column width="200" prop="seatNumber" label="座位号">
            </el-table-column>


            <el-table-column width="200" prop="status" label="状态">
            </el-table-column>

            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-popconfirm :title="('确认删除' + scope.row.floor + '楼，' + scope.row.seatNumber + '号座位吗?')"
                  @confirm="handleDelete(scope.$index, scope.row)">
                  <el-button style="margin-left: 20px" icon="el-icon-delete" size="mini" type="danger" slot="reference"
                    :disabled="!scope.row.status === '空闲'">删除</el-button>
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

  </div>
</template>

<script>
import mqtt from 'mqtt/dist/mqtt'
import { mqttUrl, mqttConfig } from '../../mqttPropertity'
export default {
  name: "index",
  data() {
    return {
      size: 5,
      current: 1,
      total: 0,
      floors: [],
      floor: '',
      seats: [],
      newSeat: {
        floor: '',
        seatNumber: ''
      },
      temperature: null,
      humidity: null,
      lamp: null
    }
  },
  watch: {
    floor(newValue, oldValue) {
      if (oldValue != '') {
        this.unsubscribeFloor(oldValue)
      }
      this.subscribeFloor(newValue);

      this.temperature = null;
      this.humidity = null;
      this.lamp = null;
    }
  },
  // 获取人脸数据
  created() {
    this.getSeatList()
    this.getFloors()
    this.initMqtt()
  },
  methods: {
    initMqtt() {
      this.MQTT = mqtt.connect(mqttUrl, mqttConfig)

      // 检测mqtt是否连接成功
      this.MQTT.on('error', (e) => {
        console.log(e)
        this.MQTT.end();
      });
      const _this = this;
      // 设置来收到消息处理的方法
      this.MQTT.on('message', function (top, message) {
        message = message.toString();
        const topic = top.split('/');
        //根据不同主题进行赋值操作
        switch (topic[0]) {
          case 'floor':
            // #36#34#off 温度 湿度 灯
            const value = message.split('#');
            // ['', '32', '36', 'off']
            _this.temperature = value[1];
            _this.humidity = value[2];
            _this.lamp = value[3];
            break;
        }
      });

    },
    subscribeFloor(id) {
      this.MQTT.subscribe("floor/" + id, { qos: 0 }, err => {
        if (!err) {
        } else {
          console.warn('订阅失败')
        }
      })
      this.MQTT.on('connect', () => {

      });
    },
    unsubscribeFloor(id) {
      this.MQTT.unsubscribe("floor/" + id, { qos: 0 }, err => {
        if (!err) {
        } else {
          console.warn('取消订阅失败')
        }
      })
      this.MQTT.on('connect', () => {

      });
    },
    switchLamp() {
      console.log(this.lamp)
      this.MQTT.publish("led/" + this.floor, this.lamp);
    },
    addSeat() {
      this.$http.post("/seat/add", this.newSeat).then(res => {

        this.$message({
          showClose: true,
          message: res.data.msg,
          type: 'success',
          duration: 3000
        })
        this.getSeatList()
        this.getFloors()
      })
    },
    handleFloorChange(val) {
      this.floor = val
      this.getSeatList()
    },
    // 分页
    handleSizeChange(val) {
      this.size = val
      this.getSeatList()
    },
    // 分页
    handleCurrentChange(val) {
      this.current = val
      this.getSeatList()
    },
    // 座位列表
    getSeatList() {
      this.$http.get('/seat/list', {
        params: {
          current: this.current,
          size: this.size,
          floor: this.floor
        }
      }).then(res => {
        this.seats = res.data.data.records
        this.size = res.data.data.size
        this.current = res.data.data.current
        this.total = res.data.data.total
      })
    },
    getFloors() {
      this.$http.get('/seat/floors').then(res => {
        this.floors = res.data.data
      })
    },
    //删除
    handleDelete(index, row) {
      console.log(index, row);
      this.$http.delete("/seat/" + row.floor + '/' + row.seatNumber).then(res => {
        this.getSeatList();
        this.$message.success(res.data.msg)
      })
    }
  }
}
</script>

<style >
.page-row {
  margin-top: 20px;
  text-align: center;
}
</style>