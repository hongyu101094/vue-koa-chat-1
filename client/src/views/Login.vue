<template>
  <div class="login">
    <message
      :type="this.message.type"
      :message="this.message.message"
    ></message>
    <message-box
      :visible="this.messageBox.visible"
      :messageBoxEvent="this.messageBox.messageBoxEvent"
      @confirm="confirm"
      :hasCancel="false"
    >
      <p slot="content">{{ this.messageBox.message }}</p>
    </message-box>
    <div class="wrapper fadeInDown">
      <!-- <img src="http://img4me.com/0dRQey.png" border="0" width="212" height="26" alt="Generated by IMG4Me" /> -->
      <div id="formContent">
        <h2 class="active">登入</h2>
        <router-link to="/signup">
          <h2 class="inactive">註冊</h2>
        </router-link>
        <div class="fadeIn first">
          <img src="../assets/icon.svg" id="icon" alt="User Icon" />
        </div>

        <form>
          <input
            type="text"
            class="fadeIn second"
            placeholder="帳號"
            v-model="name"
          />
          <input
            type="password"
            class="fadeIn third"
            placeholder="密碼"
            v-model="password"
          />
          <input
            type="button"
            @click="login"
            class="fadeIn fourth"
            value="登入"
          />
        </form>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "login",
  props: {},
  components: {},
  sockets: {
    connect: function() {
      console.log("socket to notification channel connected");
    },
    disconnect: function() {
      this.$socket.emit("disconnect");
    },
  },
  data() {
    return {
      name: "",
      password: "",
      message: {
        type: "",
        message: "",
      },
      messageBox: {
        visible: false,
        message: "", //弹窗内容
        hasCancel: true, //弹窗是否有取消键
        messageBoxEvent: "", // 弹窗事件名称
      },
    };
  },
  computed: {},
  watch: {},
  methods: {
    login() {
      if (this.name !== "" && this.password !== "") {
        axios
          .post("http://localhost:3000/api/auth/login", {
            name: this.name,
            password: this.password,
          })
          .then((res) => {
            if (res.status === 200) {
              console.log(res.data);
              // 保存soket.io
              // socket.emit("login", res.data.userInfo.user_id);
              this.$socket.emit("login", res.data.userInfo.id);
              localStorage.setItem("token", res.data.token);
              localStorage.setItem(
                "userInfo",
                JSON.stringify(res.data.userInfo)
              );
              // 設置 token 時效，毫秒為單位
              localStorage.setItem("expire", res.data.expire);
              console.log(res.data.expire);
              //弹窗
              this.messageBox.messageBoxEvent = "login";
              this.messageBox.visible = true;
              this.messageBox.message = "您已登入成功";
            } else {
              console.log(res.data.text);
              // this.$message({
              //   message: res.data.message,
              //   type: "error",
              // });
            }
          })
          .catch((err) => {
            console.log(err);
            console.log(err.response);
            this.message.type = "error";
            this.message.message = err.response.data;

            // this.$message({
            //   message: "服务器出错啦",
            //   type: "error",
            // });
          });
      } else {
        // const message = this.name === "" ? "请输入用户名" : "请输入密码";
        // this.$message({
        //   message: message,
        //   type: "warn",
        // });
      }
    },
    confirm(value) {
      console.log("confirm", value);
      if (value === "login") {
        this.messageBox.visible = false;
        this.$router.push("/");
      }
    },
  },
};
</script>
<style lang="scss" scoped>
@import "../assets/login.scss";
</style>
