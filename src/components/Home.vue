<template>
  <div class="layout">
    <Layout :style="{ minHeight: '100vh' }">
      <Sider class="sider">
        <div class="layout-logo">
          <img src="../assets/1.png" alt="" class="logo">
        </div>
        <Menu
          class="menu"
          :open-names="['1', '2', '3']"
          theme="dark"
          width="auto"
          :active-name="activeName"
        >
          <Submenu name="1">
            <template slot="title">
              <Icon type="ios-archive" />
              水质数据管理
            </template>
            <MenuItem name="/recent" to="/recent">
              <Icon type="md-bonfire" />
              近日监测
            </MenuItem>
            <MenuItem name="/manage" to="/manage">
              <Icon type="md-book" />
              历史数据
            </MenuItem>
            <MenuItem name="/trend" to="trend">
              <Icon type="md-barcode" />
              数据趋势
            </MenuItem>
          </Submenu>

          <Submenu name="2">
            <template slot="title">
              <Icon type="md-trending-up" />
              水质数据预测
            </template>
            <MenuItem name="/predict" to="predict">
              <Icon type="md-alarm" />
              下日预测
            </MenuItem>
            <MenuItem name="/train" to="train">
              <Icon type="ios-build" />
              模型训练
            </MenuItem>
          </Submenu>

          <Submenu name="3" style="display: none" id="user_menu">
            <template slot="title">
              <Icon type="ios-contact" />
              系统用户管理
            </template>
            <MenuItem name="/user" to="user">
              <Icon type="ios-archive" />
              用户管理
            </MenuItem>
          </Submenu>
        </Menu>
      </Sider>

      <Layout>
        <Header class="title" style="z-index: 999">
          <Row>
            <Col span="8" style="text-align: left; line-height: 60px"
              >贝塔渔智能养殖水质分析决策系统</Col
            >
            <Col span="4" offset="12" style="line-height: 60px">
              <Dropdown @on-click="dropDownClicked">
                <span href="javascript:void(0)">
                  {{ currentUser.username }}
                  <Icon type="ios-arrow-down"></Icon>
                </span>
                <DropdownMenu slot="list">
                  <DropdownItem name="info">用户信息</DropdownItem>
                  <DropdownItem name="editPassword">修改密码</DropdownItem>
                  <DropdownItem name="setThreshold">设置阈值</DropdownItem>
                  <DropdownItem name="logout">注销</DropdownItem>
                </DropdownMenu>
              </Dropdown>
            </Col>
          </Row>
        </Header>

        <Layout>
          <Content>
            <router-view></router-view>
          </Content>
          <Footer class="footer"> </Footer>
        </Layout>
      </Layout>
    </Layout>

    <!-- 用户信息 -->
    <Modal v-model="info_modal_flag" title="用户信息" cancel-text="">
      <Row class="editLine">
        <Col span="4" offset="3" style="text-align: left">
          <span>用户名</span>
        </Col>
        <Col span="6" offset="1">
          <span>{{ currentUser.username }}</span>
        </Col>
      </Row>
      <Row class="editLine">
        <Col span="4" offset="3" style="text-align: left">
          <span>用户角色</span>
        </Col>
        <Col span="6" offset="1">
          <span>{{ currentUser.role }}</span>
        </Col>
      </Row>
      <Row class="editLine">
        <Col span="4" offset="3" style="text-align: left">
          <span>用户权限</span>
        </Col>
        <Col span="12" offset="1">
          <span>{{ currentUser.authority }}</span>
        </Col>
      </Row>
    </Modal>

    <!-- 修改密码 -->
    <Modal
      v-model="pwd_modal_flag"
      title="修改密码"
      cancel-text="取消"
      ok-text="提交"
      @on-ok="editPassword"
    >
      <Row class="editLine">
        <Col span="4" offset="3">
          <span>原密码</span>
        </Col>
        <Col span="8" offset="1">
          <Input
            size="small"
            type="password"
            v-model="password.originPassword"
          ></Input>
        </Col>
      </Row>
      <Row class="editLine">
        <Col span="4" offset="3">
          <span>新密码</span>
        </Col>
        <Col span="8" offset="1">
          <Input
            size="small"
            type="password"
            v-model="password.newPassword"
          ></Input>
        </Col>
      </Row>
      <Row class="editLine">
        <Col span="4" offset="3">
          <span>确认密码</span>
        </Col>
        <Col span="8" offset="1">
          <Input
            size="small"
            type="password"
            v-model="password.confirmPassword"
          ></Input>
        </Col>
      </Row>
    </Modal>

    <!-- 设置阈值 -->
    <Modal
      v-model="threshold_modal_flag"
      title="设置阈值"
      cancel-text="取消"
      ok-text="提交"
      @on-ok="setThreshold"
    >
      <Row class="editLine">
        <Col span="4" offset="1" style="text-align: center">
          <span>酸碱度</span>
        </Col>
        <Col span="6">
          <Input v-model="threshold.ph"></Input>
        </Col>
        <Col span="4" offset="1" style="text-align: center">
          <span>溶解氧</span>
        </Col>
        <Col span="6">
          <Input v-model="threshold.do"></Input>
        </Col>
      </Row>

      <Row class="editLine">
        <Col span="4" offset="1" style="text-align: center">
          <span>氨氮</span>
        </Col>
        <Col span="6">
          <Input v-model="threshold.nh3N"></Input>
        </Col>
        <Col span="4" offset="1" style="text-align: center">
          <span>温度</span>
        </Col>
        <Col span="6">
          <Input v-model="threshold.waterTemperature"></Input>
        </Col>
      </Row>
    </Modal>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      activeName: this.$route.path,
      currentUser: {}, //用户信息
      password: {
        originPassword: "",
        newPassword: "",
        confirmPassword: ""
      },
      threshold: {
        ph: "", //酸碱度（PH）
        do: "", //溶解氧（DO mg/L）
        nh3N: "", //氨氮
        waterTemperature: "" //温度
      },
      info_modal_flag: false,
      pwd_modal_flag: false,
      admin_flag: false,
      threshold_modal_flag: false
    };
  },
  watch: {
    $route() {
      this.activeName = this.$route.path;
    }
  },
  created() {
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
            let user_menu = document.getElementById("user_menu");
            user_menu.style.display = "block";
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

    //弹出框
    dropDownClicked(name) {
      let _this = this;
      switch (name) {
        case "logout":
          this.logout();
          break;
        case "info":
          this.info_modal_flag = true;
          break;
        case "editPassword":
          this.pwd_modal_flag = true;
          break;
        case "setThreshold":
          _this.showThreshold();
          break;
      }
    },

    //注销
    logout() {
      let _this = this;
      this.axios.get("/logout").then(function(response) {
        if (response.data.status === "success") {
          _this.$router.replace({ path: "/" });
        } else {
          _this.$Message.error("注销失败!");
        }
      });
    },

    //修改密码
    editPassword() {
      let _this = this;
      if (this.checkPassword()) {
        this.axios
          .post(
            "/user/editPassword/" + this.currentUser.id.toString(),
            this.qs.stringify({
              originPassword: _this.password.originPassword,
              newPassword: _this.password.newPassword
            })
          )
          .then(function(response) {
            if (response.data.status === "success") {
              _this.$Message.success("密码修改成功");
            } else if (response.data.status === "error") {
              _this.$Message.error("原密码错误");
            } else {
              _this.$Message.error("密码修改失败");
            }
          });
      }
      this.password.originPassword = "";
      this.password.newPassword = "";
      this.password.confirmPassword = "";
    },

    //检验密码
    checkPassword() {
      if (
        this.password.originPassword === "" ||
        this.password.newPassword === "" ||
        this.password.confirmPassword === ""
      ) {
        this.$Message.error("输入不得为空");
        return false;
      } else if (this.password.newPassword !== this.password.confirmPassword) {
        this.$Message.error("新密码不一致");
        return false;
      }
      return true;
    },

    //回显阈值
    showThreshold() {
      // console.log("111")
      this.threshold_modal_flag = true;
      let _this = this;
      this.axios.get("/warning/findByUser").then(function(response) {
        if (response.data.status === "success") {
          _this.threshold.ph = response.data.ph;
          _this.threshold.do = response.data.do;
          _this.threshold.nh3N = response.data.nh3N;
          _this.threshold.waterTemperature = response.data.waterTemperature;
        }
      });
    },

    //设置阈值
    setThreshold() {
      let _this = this;
      if (this.checkThreshold()) {
        this.axios
          .post(
            `/warning/save?uid=${_this.currentUser.id}&PH=${_this.threshold.ph}&DO=${_this.threshold.do}&NH3N=${_this.threshold.nh3N}&WaterTemperature=${_this.threshold.waterTemperature}`,
            this.qs.stringify({
              id: _this.currentUser.id,
              PH: _this.threshold.ph,
              DO: _this.threshold.do,
              NH3N: _this.threshold.nh3N,
              waterTemperature: _this.threshold.waterTemperature
            })
          )
          .then(function(response) {
            if (response) {
              _this.$Message.success("阈值设置成功");
            } else {
              _this.$Message.error("阈值设置失败");
            }
          });
      }
    },

    //检验输入阈值
    checkThreshold() {
      if (
        this.threshold.ph === "" ||
        this.threshold.do === "" ||
        this.threshold.nh3N === "" ||
        this.threshold.waterTemperature === ""
      ) {
        this.$Message.error("输入不得为空");
        return false;
      }
      return true;
    }
  }
};
</script>

<style scoped>
.title {
  font-size: 20px;
  color: #ffffff;
  background-color: #6665e0;
  opacity: 0.9;
  height: 60px;
}

.sider {
  /* background-color: rgb(245, 238, 238); */
}

.menu {
  /* background-color: #e8e8e8; */
}

.logo{
  width: 80px;
  margin: 5px;
}

.editLine {
  font-size: 16px;
  line-height: 24px;
  margin-top: 10px;
  margin-bottom: 10px;
}
</style>
