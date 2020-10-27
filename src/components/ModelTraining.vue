<template>
  <!-- 模型训练 -->
  <div class="content">
    <div class="query">
      <Row class="queryline">
        <Col span="2" offset="1" style="text-align: center">水质因素</Col>
        <Col span="3" style="text-align: left">
          <Select v-model="indicator">
            <Option value="ph">酸碱度</Option>
            <Option value="do">溶解氧</Option>
            <Option value="nh3N">氨氮</Option>
            <Option value="waterTemperature">温度</Option>
          </Select>
        </Col>

        <Col span="3" offset="1" style="text-align: center">模型</Col>
        <Col span="3" style="text-align: left">
          <Select v-model="method">
            <Option value="SVM">SVM</Option>
            <Option value="Adaboost">Adaboost</Option>
            <Option value="Xgboost">Xgboost</Option>
            <Option value="BP">BP</Option>
            <Option value="LSTM">LSTM</Option>
            <Option value="DecesionTree">DecesionTree</Option>
          </Select>
        </Col>

        <Col span="3" offset="2">
          <Button
            type="primary"
            shape="circle"
            icon="ios-build"
            long
            @click="trainModel"
            >训练模型</Button
          >
        </Col>
      </Row>
    </div>

    <div id="plot_holder" v-if="plot_loading_flag" style="height: 600px">
      <spin size="large" fix v-if="plot_loading_flag" style="font-size: 20px"
        >{{ method }}模型训练中，请稍等片刻...</spin
      >
    </div>

    <div id="plot" style="height: 600px"></div>
  </div>
</template>

<script>
export default {
  name: "ModelTraining",
  data() {
    return {
      indicator: "ph",
      method: "LSTM",
      rmse: "",
      pred: [],
      real: [],
      plot_loading_flag: false,
      currentUser: {}
    };
  },
  created() {
    this.getCurrentUser();
  },

  methods: {
    //获取数据
    trainModel() {
      let _this = this;
      let charts = document.getElementById("plot");
      charts.style.visibility = "hidden";
      charts.style.height = "0";
      _this.plot_loading_flag = true;
      this.axios
        .get("model/training", {
          params: {
            indicator: _this.indicator,
            method: _this.method,
            threshold: _this.threshold,
            uid: _this.currentUser.id
          }
        })
        .then(function(response) {
          let body = response.data;
          if (body.status === "success") {
            _this.rmse = body.data.rmse;
            _this.pred = body.data.pred;
            _this.real = body.data.real;
            charts.style.height = "600px";
            _this.plot();
          } else if (body.status === "deny") {
            _this.$Message.error("权限不足，请联系管理员");
            charts.style.height = "600px";
            charts.style.visibility = "visible";
            _this.plot_loading_flag = false;
          } else {
            _this.$Message.error("训练失败！");
            charts.style.height = "600px";
            charts.style.visibility = "visible";
            _this.plot_loading_flag = false;
          }
        });
    },

    getCurrentUser() {
      let _this = this;
      this.axios.get("/user/current").then(function(response) {
        _this.currentUser = response.data;
      });
    },

    plot() {
      let _this = this;
      let chart = this.echarts.init(document.getElementById("plot"));
      let x = new Array(_this.pred.length);
      for (var i = 0; i < _this.pred.length; i++) {
        x[i] = i + 1;
      }

      var colors = ["#5793f3", "#d14a61", "#675bba"];

      let option = {
        title: {
          show: true,
          left: "10%",
          text:
            "预测" +
            _this.indicator.toUpperCase() +
            "的" +
            _this.method +
            "模型训练结果",
          subtext: "测试集上均方根误差为" + _this.rmse,
          textStyle: {
            fontSize: "17px"
          }
        },
        color: colors,
        tooltip: {
          trigger: "item",
          axisPointer: {
            type: "cross"
          }
        },
        legend: {
          data: ["预测值", "实际值"]
        },
        grid: {
          top: 70,
          bottom: 50
        },
        xAxis: [
          {
            type: "category",
            axisTick: {
              alignWithLabel: true
            },

            axisLine: {
              onZero: false,
              lineStyle: {
                color: colors[0]
              }
            },
            axisPointer: {
              label: {
                formatter: function(params) {
                  return (
                    "预测值  " +
                    params.value +
                    (params.seriesData.length
                      ? "：" + params.seriesData[0].data
                      : "")
                  );
                }
              }
            },
            data: x
          },
          {
            type: "category",
            axisTick: {
              alignWithLabel: true
            },
            axisLine: {
              onZero: false,
              lineStyle: {
                color: colors[1]
              }
            },
            axisPointer: {
              label: {
                formatter: function(params) {
                  return (
                    "实际值  " +
                    params.value +
                    (params.seriesData.length
                      ? "：" + params.seriesData[0].data
                      : "")
                  );
                }
              }
            },
            data: x
          }
        ],
        yAxis: [
          {
            type: "value",
            scare: true,
            // interval: 0.5
          }
        ],
        series: [
          {
            name: "预测值",
            type: "line",
            xAxisIndex: 1,
            smooth: true,
            data: _this.pred
          },
          {
            name: "实际值",
            type: "line",
            smooth: true,
            data: _this.real
          }
        ]
      };

      chart.setOption(option);
      var charts = document.getElementById("plot");
      charts.style.visibility = "visible";
      _this.plot_loading_flag = false;
    }

    //折线图绘制
    /* plot() {
      let option = {

        tooltip: {
          trigger: "item"
        },
        xAxis: {
          type: "category",
          data: x
        },
        yAxis: {
          type: "value",
          scale: true
        },
        series: [
          {
            data: _this.pred,
            itemStyle: {
              normal: {
                // color: _this.pointColors,
                color: function(param) {
                  if (param.value > _this.threshold) {
                    return "purple";
                  } else {
                    return "aqua";
                  }
                },
                lineStyle: {
                  color: "aqua"
                }
              }
            },
            type: "line"
          },
          {
            data: _this.real,
            itemStyle: {
              normal: {
                color: "red",
                lineStyle: {
                  color: "red"
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
    } */
  }
};
</script>

<style scoped>
.query {
  margin-top: 20px;
  margin-bottom: 20px;
}

.queryline {
  line-height: 32px;
}

#plot_holder {
  position: relative;
  margin-top: 20px;
  margin-left: 30px;
  margin-right: 30px;
}
</style>
