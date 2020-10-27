<template>
  <!-- 下日预测 -->
  <div class="content">
    <div class="query">
      <Row class="queryline">
        <!-- 水质指标 选择 -->
        <Col span="2" offset="1" style="text-align: center">水质因素</Col>
        <Col span="3" style="text-align: left">
          <Select v-model="indicator" @on-change="getAllMethods">
            <Option value="ph">酸碱度</Option>
            <Option value="do">溶解氧</Option>
            <Option value="nh3N">氨氮</Option>
            <Option value="waterTemperature">温度</Option>
          </Select>
        </Col>

        <!-- 机器模型 选择 -->
        <Col span="3" offset="1" style="text-align: center">模型</Col>
        <Col span="3" style="text-align: left">
          <Select v-model="method" @on-change="getAvailableMethods">
            <Option value="all">全部</Option>
            <Option v-for="item in modelList" :value="item" :key="item">{{
              item
            }}</Option>
          </Select>
        </Col>
      </Row>
    </div>

    <!-- 可用模型列表 -->
    <div class="data_table">
      <div class="table_title">可用模型列表</div>
      <Table
        class="table"
        :loading="model_table_flag"
        border
        :columns="columns"
        :data="availableModels"
      >
        <template slot-scope="{ row, index }" slot="action">
          <Button
            type="success"
            size="default"
            @click="getNextMonthPrediction(index)"
            >预测</Button
          >
          <Button type="error" size="default" @click="deleteModel(index)"
            >删除</Button
          >
        </template>
      </Table>
    </div>

    <div id="plot_holder" v-if="plot_loading_flag" style="height: 400px">
      <spin size="large" fix v-if="plot_loading_flag" style="font-size: 20px"
        >使用{{ chosenModel.method }}预测{{
          indicator.toUpperCase()
        }}中...</spin
      >
    </div>

    <div id="plot" style="height: 400px"></div>
  </div>
</template>

<script>
export default {
  name: "WaterPrediction",
  data() {
    return {
      indicator: "ph",
      method: "all",
      modelList: [], //机器学习模型种类
      availableModels: [],
      prediction: "",
      chosenModel: {},
      model_table_flag: false,
      plot_loading_flag: false,
      plotWaterQualities: [],
      plotDates: [], //x轴数据
      plotWaterQualitiesSecond: [],
      currentUser: {}, //用户信息
      warningValue: 0, //阈值
      columns: [
        {
          title: "模型类型",
          key: "method",
          align: "center"
        },
        {
          title: "均方根误差",
          key: "rmse",
          align: "center",
          sortable: "true"
        },
        {
          title: "训练者",
          key: "user",
          align: "center"
        },
        {
          title: "训练日期",
          key: "date",
          align: "center",
          sortable: "true"
        },
        {
          title: "操作",
          slot: "action",
          align: "center"
        }
      ]
    };
  },
  mounted() {
    this.getAllMethods();
    this.getCurrentUser();
  },
  methods: {
    //获取用户信息
    getCurrentUser() {
      let _this = this;
      this.axios.get("/user/current").then(function(response) {
        if (response.data.status === "noLogin") {
          _this.$router.replace({ path: "/" });
        } else {
          _this.currentUser = response.data;
          if (_this.currentUser.role.id === 1) {
            _this.currentUser.role = "超级管理员";
            _this.currentUser.authority = "全部权限";
          } else if (_this.currentUser.role.id === 2) {
            _this.currentUser.role = "管理员";
            _this.currentUser.authority =
              "水质数据查询、增加、修改、删除、预测; 模型训练";
          } else {
            _this.currentUser.role = "普通用户";
            _this.currentUser.authority = "水质数据查询、预测";
          }
        }
      });
    },

    //获取列表信息
    getAvailableMethods() {
      let _this = this;
      this.model_table_flag = true;
      var charts = document.getElementById("plot");
      charts.style.height = "0";
      charts.style.visibility = "hidden";
      this.axios
        .get("/model/available", {
          params: {
            indicator: _this.indicator,
            method: _this.method
          }
        })
        .then(function(response) {
          _this.availableModels = response.data;
          for (var i = 0; i < _this.availableModels.length; i++) {
            _this.availableModels[i].user =
              _this.availableModels[i].user.username;
            _this.availableModels[i].date = new Date(
              _this.availableModels[i].date
            ).Format("yyyy-MM-dd hh:mm:ss");
          }
          _this.model_table_flag = false;
        });
    },

    //显示所有的 可用模型信息
    getAllMethods() {
      let _this = this;
      this.model_table_flag = true;
      var charts = document.getElementById("plot");
      charts.style.height = "0";
      charts.style.visibility = "hidden";
      this.axios
        .get("model/list", {
          params: {
            indicator: _this.indicator
          }
        })
        .then(function(response) {
          _this.modelList = response.data;
          _this.method = "all";
          _this.getAvailableMethods();
        });
    },

    //点击 删除模型
    deleteModel(index) {
      let _this = this;
      this.chosenModel = this.availableModels[index];
      this.axios
        .post("model/delete/" + this.chosenModel.id.toString())
        .then(function(response) {
          if (response.data.status === "success") {
            _this.$Message.success("删除成功");
            _this.getAvailableMethods();
          } else if (response.data.status === "deny") {
            _this.$Message.error("权限不足，请联系管理员");
          } else {
            _this.$Message.error("删除失败");
          }
        });
    },

    //点击  下日预测
    getNextMonthPrediction(index) {
      let _this = this;
      this.plot_loading_flag = true;
      let charts = document.getElementById("plot");
      charts.style.height = "0";
      charts.style.visibility = "hidden";
      this.chosenModel = this.availableModels[index];
      this.axios
        .get("model/prediction", {
          params: {
            id: _this.chosenModel.id,
            indicator: _this.indicator,
            method: _this.method,
            uid: _this.currentUser.id
          }
        })
        .then(function(response) {
          let body = response.data;
          if (body.status === "success") {
            _this.prediction = body.pred;
            _this.plotWaterQualities = body.forPlot;
            _this.plotWaterQualitiesSecond = [
              "-",
              "-",
              "-",
              "-",
              _this.plotWaterQualities[4],
              _this.prediction
            ];
            _this.plotWaterQualities[5] = "-";
            _this.plotDates = body.dates;
            _this.warningValue = body.warningValue;
            charts.style.height = "400px";
            _this.plot();
          } else {
            charts.style.visibility = "visible";
            charts.style.height = "400px";
            _this.plot_loading_flag = false;
            _this.$Message.error("预测失败！");
          }
        });
    },
    //下日预测显示   折线图
    plot() {
      let _this = this;
      let chart = this.echarts.init(document.getElementById("plot"));
      // _this.warningValue = 1;
      console.log(_this.warningValue);
      let option = {
        title: {
          left: "center",
          text:
            _this.indicator.toUpperCase() + "下日预测值: " + _this.prediction,
          subtext:
            "由管理员" +
            _this.chosenModel.user +
            "在" +
            _this.chosenModel.date +
            "训练的" +
            _this.chosenModel.method +
            "模型预测"
        },
        tooltip: {
          trigger: "item"
        },
        xAxis: {
          type: "category",
          data: _this.plotDates
        },
        yAxis: {
          type: "value",
          scale: true
        },
        series: [
          {
            data: _this.plotWaterQualities,
            symbolSize: 4,
            itemStyle: {
              normal: {
                color: "blue",
                lineStyle: {
                  color: "blue"
                }
              }
            },
            type: "line"
          },
          {
            data: _this.plotWaterQualitiesSecond,
            smooth: true,
            // symbolSize: 4,
            symbol: "circle",
            itemStyle: {
              normal: {
                color: function(param) {
                  if (param.value > _this.warningValue) {
                    return "purple";
                  } else {
                    return "green";
                  }
                },
                lineStyle: {
                  color: "green"
                },
                borderWidth: function(param) {
                  if (param.value > _this.warningValue) {
                    return 2;
                  } else {
                    return 1;
                  }
                }
              }
            },
            type: "line"
          }
        ]
      };
      chart.setOption(option);
      var charts = document.getElementById("plot");
      charts.style.visibility = "visible";
      _this.plot_loading_flag = false;
    }
  }
};
</script>

<style scoped>
.query {
  margin-top: 20px;
}

.queryline {
  line-height: 32px;
}

.data_table {
  margin-top: 20px;
  margin-bottom: 15px;
  padding-left: 30px;
  padding-right: 30px;
}

.table_title {
  margin-bottom: 10px;
  font-size: 16px;
}

.plot_title {
  margin-bottom: 15px;
  font-size: 18px;
}

#plot_holder {
  position: relative;
  margin-left: 30px;
  margin-right: 30px;
}

/*#plot {*/
/*position: relative;*/
/*}*/
</style>
