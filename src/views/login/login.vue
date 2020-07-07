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
  <div class="login-bg">
    <div class="login-head"></div>
    <div class="login-lang">
      <lang-select class="right-menu-item hover-effect"></lang-select>
    </div>
    <div class="login">
      <div class="login-content">
        <template>
          <el-form ref="loginForm" :model="loginForm" :rules="rules" class="login-form">
            <el-form-item :label="$t('login.user')" prop="user">
              <el-input v-model="loginForm.user" :placeholder="$t('inputText.user')"></el-input>
            </el-form-item>
            <el-form-item :label="$t('login.password')" prop="password">
              <el-input
                v-model="loginForm.password"
                :placeholder="$t('inputText.password')"
                type="password"
              ></el-input>
            </el-form-item>
            <el-form-item :label="$t('login.verificationCode')" prop="vercode">
              <div style="width: 100%;">
                <el-input
                  style="width: 250px;"
                  v-model="loginForm.vercode"
                  :placeholder="$t('inputText.verificationCode')"
                  @keyup.enter.native="submit('loginForm')"
                ></el-input>
                <span class="codeUrlImg">
                  <img style="width: 100%;height: 100%;" :src="codeUrl" alt @click="changeCode()" />
                </span>
              </div>
            </el-form-item>
            <el-form-item style="width:100%;text-align:center;">
              <el-button
                type="primary"
                style="width:100%;"
                @click="submit('loginForm')"
                :loading="logining"
              >{{this.$t('login.login')}}</el-button>
            </el-form-item>
          </el-form>
        </template>
      </div>
    </div>
    <div class="login-footer">浙江网新恩普软件有限公司 客服热线：400-000-9790 www.epsoft.com.cn</div>
  </div>
</template>
<script>
import {
  login,
  networkList,
  haveNode,
  getPictureCheckCode,
  encryption
} from "@/util/api";
import url from "@/util/url";
import router from "@/router";
import bg from "@/../static/image/banner.png";
import logo from "@/../static/image/logo-2 copy@1.5x.jpg";
import { delCookie } from "@/util/util";
import errcode from "@/util/errcode";
const sha256 = require("js-sha256").sha256;
const gm = require("@/util/SM2Sign");
import utils from "@/util/sm_sha";
import langSelect from "@/components/LangSelect";
export default {
  name: "login",
  components: {
    "lang-select": langSelect
  },
  data: function() {
    return {
      bgLogin: bg,
      logoPng: logo,
      logining: false,
      codeUrl: url.codeUrl,
      loginForm: {
        user: "",
        password: "",
        vercode: ""
      },
      authToken: null,
      encryption: "hash"
    };
  },
  computed: {
    rules() {
      var obj = {
        user: [
          {
            required: true,
            message: this.$t("inputText.user"),
            trigger: "blur"
          }
        ],
        password: [
          {
            required: true,
            message: this.$t("inputText.password"),
            trigger: "blur"
          }
        ],
        vercode: [
          {
            required: true,
            message: this.$t("inputText.verificationCode"),
            trigger: "blur"
          }
        ]
      };
      return obj;
    }
  },
  mounted: function() {
    this.changeCode();
    this.getEncryption();
  },
  methods: {
    handleEncryption: function() {
      if (this.encryption == "guomi") {
        localStorage.setItem("encryptionId", 1);
      } else {
        localStorage.setItem("encryptionId", 0);
      }
    },
    submit: function(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.logining = true;
          this.userLogin();
        } else {
          return false;
        }
      });
    },
    changeCode: function() {
      this.codeUrl = "";
      this.authToken = "";
      getPictureCheckCode()
        .then(res => {
          if (res.data.code === 0) {
            this.codeUrl = `data:image/png;base64,${res.data.data.base64Image}`;
            this.authToken = res.data.data.token;
          } else {
            this.codeUrl = "";
            this.authToken = "";
            this.$message({
              message: this.$chooseLang(res.data.code),
              type: "error",
              duration: 2000
            });
          }
        })
        .catch(err => {
          this.codeUrl = "";
          this.authToken = "";
          this.$message({
            message: this.$t("text.systemError"),
            type: "error",
            duration: 2000
          });
        });
    },
    userLogin: function(callback) {
      delCookie("JSESSIONID");
      delCookie("NODE_MGR_ACCOUNT_C");
      let reqData = {
        account: this.loginForm.user,
        accountPwd: sha256(this.loginForm.password)
      };
      let checkCode = this.loginForm.vercode;
      login(reqData, checkCode, this.authToken)
        .then(res => {
          if (res.data.code === 0) {
            localStorage.setItem("groupName", "");
            localStorage.setItem("groupId", "");
            localStorage.setItem("folderList", "");
            localStorage.setItem("user", res.data.data.account);
            localStorage.setItem("root", res.data.data.roleName);
            localStorage.setItem("token", res.data.data.token);
            sessionStorage.setItem(
              "accountStatus",
              res.data.data.accountStatus
            );
            sessionStorage.setItem("reload", 1);
            router.push("/main");
          } else {
            this.changeCode();
            this.loginForm.password = "";
            this.logining = false;
            let msgErrorContent =
              this.$chooseLang(res.data.code) || this.$t("text.loginFail");
            this.$message.error(msgErrorContent);
          }
        })
        .catch(err => {
          this.changeCode();
          this.loginForm.password = "";
          this.logining = false;
          let msgErrorContent = this.$t("text.reqOvertime");
          this.$message.error(msgErrorContent);
        });
    },
    getEncryption: function() {
      encryption()
        .then(res => {
          if (res.data.code === 0) {
            if (res.data.data == 1) {
              this.encryption = "guomi";
            } else {
              this.encryption = "hash";
            }
            localStorage.setItem("encryptionId", res.data.data);
          } else {
            this.$message({
              message: this.$chooseLang(res.data.code),
              type: "error",
              duration: 2000
            });
          }
        })
        .catch(err => {
          this.$message({
            message: this.$t("text.systemError"),
            type: "error",
            duration: 2000
          });
        });
    }
  }
};
</script>
<style scoped>
.login-label.is-required:not(.is-no-asterisk) > .el-form-item__label:before {
  content: "" !important;
}
.login-bg {
  position: relative;
  width: 100%;
  height: 100%;
  background-repeat: no-repeat;
  background-size: 100% 100%;
}
.login-head {
  width: 62.5%;
  height: 12.9%;
  margin: 0 auto;
  background: url("../../assets/images/logo.png") no-repeat left center;
}
.login {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  width: 100%;
  height: 74.2%;
  padding-right: 180px;
  box-sizing: border-box;
  background: #007be3 url("../../assets/images/loginbg.png") bottom center
    no-repeat;
}
.login-content {
  flex: 0 0 auto;
  width: 410px;
  box-sizing: border-box;
  padding: 20px 30px 10px;
  border-radius: 8px;
  background: #fff;
  text-align: center;
  font-size: 12px;
  color: #979faa;
  letter-spacing: 0.02px;
}
.codeUrlImg {
  display: inline-block;
  height: 36px;
  width: 84px;
  line-height: 36px;
  border: 1px solid #dcdfe6;
  border-radius: 2px;
  vertical-align: middle;
  cursor: pointer;
  text-align: center;
}
.logo {
  width: 120px;
  padding-top: 80px;
}
.login-title {
  padding: 16px 0 0px 0;
  font-size: 20px;
  color: #2e2e2e;
  letter-spacing: 0.04px;
}
.login-label {
  position: relative;
  padding: 0 60px;
}
.login-label input {
  height: 54px;
  padding-left: 60px;
}
.icon {
  width: 142px;
  height: 142px;
  fill: currentColor;
  overflow: hidden;
}
.el-form-item.is-required:not(.is-no-asterisk) > .el-form-item__label:before {
  content: "";
}
.login-form >>> .el-form-item__label {
  display: block;
  line-height: 32px;
  float: none;
  text-align: left;
}
.login-lang {
  position: absolute;
  right: 10px;
  top: 20px;
  min-width: 160px;
}
.login-encrypt {
  position: absolute;
  right: 10px;
  top: 60px;
  min-width: 160px;
}
.login-encrypt .el-radio-button__inner {
  display: inline-block;
  width: 80px;
}
.login-footer {
  margin-top: 3%;
  text-align: center;
  font-size: 16px;
}
</style>