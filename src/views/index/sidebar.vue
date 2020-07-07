/*
 * Copyright 2014-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
  <div style="height: 100%;">
    <div style="height: 100%;" class="sidebar-content">
      <div class="image-flex justify-center center logo-fixed" v-if="menuShowC">
        <span class="sidebar-contract-icon">
          <i class="el-icon-caret-left" @click="hideMune(true)" style="font-size: 18px;color:#fff;"></i>
        </span>
      </div>
      <div class="mini-sidebar-contract-icon" v-if="!menuShowC" style="height:56px;background-color: #0067e6">
        <i class="el-icon-caret-right" @click="hideMune(false)" style="font-size: 18px;color:#fff;"></i>
      </div>
      <div class="menu-content">
        <el-menu
          default-active="999"
          router
          class="el-menu-vertical-demo"
          background-color="#eee"
          @select="select"
          :collapse="!menuShowC"
          @open="handleOpen"
          @close="handleClose"
        >
          <template v-for="(item,index) in routesListC" v-if="item.menuShow">
            <el-submenu v-if="!item.leaf" :index="`${index}`" ref="ele">
              <template slot="title">
                <div :style="{'padding-left':  menuShowC ? '10px':''}">
                  <i :class="item.iconCls"></i>
                  <span :class="{'font-color-37eef2': activeIndex == index}">{{item.name}}</span>
                </div>
              </template>

              <el-menu-item
                v-for="term in item.children"
                :key="term.path"
                :index="term.path"
                v-if="term.menuShow"
              >
                <span>{{term.name}}</span>
              </el-menu-item>
            </el-submenu>
            <el-menu-item
              v-else-if="item.leaf&&item.children&&item.children.length"
              :index="item.children[0].path"
              style="width: 200px;padding-left:30px;">
              <i :class="item.iconCls"></i>
              <span slot="title">{{item.children[0].name}}</span>
            </el-menu-item>
          </template>
        </el-menu>
      </div>
    </div>
  </div>
</template>

<script>
import router from "@/router";
export default {
  name: "sidebar",
  props: ["minMenu"],
  data() {
    return {
      activeIndex: 0,
      activeRoute: "",
      userRole: localStorage.getItem("root"),
      routesList: []
    };
  },
  computed: {
    routesListC() {
      var list = this.routesList;
      list.forEach(item => {
        switch (item.nameKey) {
          case "contractTitle":
            item.name = this.$t("title.contractTitle");
            break;
          case "systemManager":
            item.name = this.$t("title.systemManager");
            break;
          case "systemMonitor":
            item.name = this.$t("title.systemMonitor");
            break;
          case "transactionAudit":
            item.name = this.$t("title.transactionAudit");
            break;
          case "subscribeEvent":
            item.name = this.$t("title.subscribeEvent");
            break;
        }
        if (item.children) {
          item.children.forEach(it => {
            switch (it.nameKey) {
              case "dataOverview":
                it.name = this.$t("title.dataOverview");
                break;
              case "nodeTitle":
                it.name = this.$t("title.nodeTitle");
                break;
              case "groupManagement":
                it.name = this.$t("title.groupManagement");
                break;
              case "PrivateKey":
                it.name = this.$t("title.PrivateKey");
                break;
              case "accountManagement":
                it.name = this.$t("title.accountManagement");
                break;
              case "contractIDE":
                it.name = this.$t("title.contractIDE");
                break;
              case "abiList":
                it.name = this.$t("title.abiList");
                break;
              case "parseAbi":
                it.name = this.$t("title.parseAbi");
                break;
              case "contractList":
                it.name = this.$t("title.contractList");
                break;
              case "CNSmanager":
                it.name = this.$t("title.CNSmanager");
                break;
              case "CRUDServiceManagement":
                it.name = this.$t("title.CRUDServiceManagement");
                break;
              case "permission":
                it.name = this.$t("title.permission");
                break;
              case "configManager":
                it.name = this.$t("title.configManager");
                break;
              case "certificate":
                it.name = this.$t("title.certificate");
                break;
              case "nodesMonitor":
                it.name = this.$t("title.nodesMonitor");
                break;
              case "hostMonitor":
                it.name = this.$t("title.hostMonitor");
                break;
              case "userTransaction":
                it.name = this.$t("title.userTransaction");
                break;
              case "unusualUser":
                it.name = this.$t("title.unusualUser");
                break;
              case "unusualContract":
                it.name = this.$t("title.unusualContract");
                break;
              case "emailAlarm":
                it.name = this.$t("title.emailAlarm");
                break;
              case "emailAlarmType":
                it.name = this.$t("title.emailAlarmType");
                break;
              case "blockEvent":
                it.name = this.$t("title.blockEvent");
                break;
              case "contractEvent":
                it.name = this.$t("title.contractEvent");
                break;
            }
          });
        }
      });
      return list;
    },
    menuShowC() {
      if (this.minMenu) {
        return this.minMenu;
      } else {
        return false;
      }
    }
  },
  mounted: function() {
    this.$nextTick(function() {
      localStorage.setItem("sidebarHide", false);
      this.changeRouter();
    });
  },
  methods: {
    changeRouter: function() {
      let list = this.$router.options.routes;
      list.forEach(item => {
        if (item.name === "帐号管理") {
          item.menuShow = false;
        }
      });
      list.forEach(item => {
        if (this.userRole === "admin" && item.name === "帐号管理") {
          item.menuShow = true;
        }
      });
      this.routesList = list;
    },
    select: function(index, indexPath) {
      this.activeIndex = indexPath[0];
      this.activeRoute = index;
    },
    handleOpen(key, keyPath) {},
    handleClose(key, keyPath) {},
    hideMune: function(val) {
      this.$emit("sidebarChange", val);
      if (this.menuShow) {
        this.menuShow = false;
      } else {
        this.menuShow = true;
      }
      if (this.$route.path === "contract" && val) {
        localStorage.setItem("sidebarHide", true);
      } else {
        localStorage.setItem("sidebarHide", false);
      }
    },
    getAdmin: function() {}
  }
};
</script>

<style scoped>
.sidebar-content {
  position: relative;
}
.logo-fixed {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9999;
  height: 56px;
  width: 200px;
  background: #0067e6 url("../../assets/images/Insigma-logo-white.png")
    no-repeat center;
}
.menu-content {
  position: absolute;
  top: 56px;
  left: 0;
  right: 0;
  bottom: 0;
  overflow-x: hidden;
  overflow-y: auto;
  background-color: #eee;
}
.el-menu-vertical-demo {
  border: none;
}
.el-menu-vertical-demo >>> .el-menu-item {
  font-size: 14px;
  text-align: left;
}
.el-menu-vertical-demo >>> .el-submenu__title {
  padding-left: 33px;
}
.el-menu-item-group > ul > .el-menu-item {
  font-size: 14px;
  text-align: left;
  padding-left: 57px !important;
  height: 46px;
  line-height: 46px;
}
.sidebar-content >>> .el-menu--collapse {
  width: 56px;
}
.sidebar-content >>> .el-menu--collapse .is-active .el-tooltip {
  padding-left: 17px !important;
  background-color: #1e293e;
}
.mini-sidebar-contract-icon {
  position: relative;
  text-align: center;
}
.mini-sidebar-contract-icon i {
  position: absolute;
  top: 20px;
  right: 10px;
  z-index: 9999;
  cursor: pointer;
}

.image-flex {
  display: -webkit-box !important;
  display: -webkit-flex !important;
  display: -ms-flexbox !important;
  display: flex !important;
  -webkit-flex-wrap: wrap;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
}
.image-flex,
.image-flex *,
.image-flex :after,
.image-flex :before {
}
.image-flex.justify-center {
  -webkit-box-pack: center;
  -webkit-justify-content: center;
  -ms-flex-pack: center;
  justify-content: center;
}
.image-flex.center {
  -webkit-box-pack: center;
  -webkit-justify-content: center;
  -ms-flex-pack: center;
  justify-content: center;
  -webkit-box-align: center;
  -webkit-align-items: center;
  -ms-flex-align: center;
  align-items: center;
}

.sidebar-icon {
  font-size: 15px;
  padding-right: 5px;
}
.sidebar-contract-icon {
  position: absolute;
  display: inline-block;
  left: 180px;
  top: 18px;
  font-size: 12px;
  letter-spacing: 0;
  text-align: right;
  cursor: pointer;
  z-index: 6666;
}
.sidebar-contract-icon i {
  cursor: pointer;
}
</style>
