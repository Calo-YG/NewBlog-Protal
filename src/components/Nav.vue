<template>
  <div>
    <div v-if="!isMobile" class="nav">
      <div class="nav-content">
        <el-row :gutter="20">
          <el-col :span="3">
            <router-link to="/">
              <img class="logo" src="../assets/logo.jpg" alt="BiaoChenXuYing" />
            </router-link>
          </el-col>
          <el-col :span="16">
            <el-menu
              :router="true"
              :default-active="activeIndex"
              active-text-color="#409eff"
              class="el-menu-demo"
              mode="horizontal"
              @select="handleSelect"
            >
              <el-menuItem
                :route="l.path"
                :index="l.index"
                v-for="l in list"
                :key="l.index"
              >
                {{ l.name }}
              </el-menuItem>
            </el-menu>
          </el-col>
          <el-col v-if="userInfo._id" :span="5">
            <div class="nav-right">
              <el-dropdown @command="handleLogout">
                <span class="el-dropdown-link">
                  {{ userInfo.name
                  }}<i class="el-icon-arrow-down el-icon--right"></i>
                </span>
                <img
                  v-if="!userInfo.avatar"
                  class="user-img"
                  src="../assets/user.png"
                  alt="BiaoChenXuYing"
                />
                <img
                  v-if="userInfo.avatar"
                  class="user-img"
                  :src="userInfo.avatar"
                  alt="BiaoChenXuYing"
                />
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="logout">登 出</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </div>
          </el-col>
          <el-col v-else :span="4">
            <div class="nav-right">
              <el-button
                size="small"
                type="primary"
                @click="handleClick('login')"
                >登录</el-button
              >
              <el-button
                size="small"
                type="danger"
                @click="handleClick('register')"
                >注册</el-button
              >
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
    <div v-else class="nav">
      <div class="nav-mobile">
        <div class="nav-mobile-logo">
          <router-link to="/">
            <img
              class="logo fl"
              src="../assets/logo.jpg"
              alt="BiaoChenXuYing"
            />
          </router-link>
        </div>
        <div class="title">{{ title }}</div>
        <div class="menu" @click="handleMenu"><i class="el-icon-menu"></i></div>
      </div>
      <div
        v-if="isShow"
        class="nav-mobile-content"
        :class="{
          'enter-slideUp': enterSlideUp,
          'leave-slideDown': leaveSlideDown,
        }"
      >
        <div class="list">
          <div @click="handleClickMenu('')" class="item">
            <router-link to="/">首 页</router-link>
          </div>
          <div @click="handleClickMenu('/articles')" class="item">
            <router-link to="/articles">文 章</router-link>
          </div>
          <div @click="handleClickMenu('/archive')" class="item">
            <router-link to="/archive">归 档</router-link>
          </div>
          <div @click="handleClickMenu('/project')" class="item">
            <router-link to="/project">项 目</router-link>
          </div>
          <div @click="handleClickMenu('/timeline')" class="item">
            <router-link to="/timeline">历 程</router-link>
          </div>
          <div @click="handleClickMenu('/message')" class="item">
            <router-link to="/message">留 言</router-link>
          </div>
          <div @click="handleClickMenu('/about')" class="item">
            <router-link to="/about">关 于</router-link>
          </div>
          <div @click="handleClickMenu('/login')" class="item">
            <span v-if="userInfo._id">{{ userInfo.name }}</span>
            <span v-else>登 录</span>
          </div>
          <div
            v-if="!userInfo._id"
            @click="handleClickMenu('/register')"
            class="item"
          >
            注 册
          </div>
          <div
            v-if="userInfo._id"
            @click="handleClickMenu('/logout')"
            class="item"
          >
            登 出
          </div>
        </div>
      </div>
    </div>
    <div
      v-if="isShow"
      class="mask"
      :class="{ 'mask-fade-out': leaveSlideDown }"
      @click="handleHideMenu"
    ></div>
    <RegisterAndLogin
      :visible="visible"
      :isMobile="isMobile"
      :handleFlag="handleFlag"
      @ok="handleOk"
      @cancel="handleCancel"
    ></RegisterAndLogin>
  </div>
</template>

<script lang="ts" setup>
import {
  defineComponent,
  defineAsyncComponent,
  ref,
  computed,
  watch,
} from "vue";
import service from "../utils/https";
import urls from "../utils/urls";
import { useStore } from "vuex";
import { useRoute, useRouter } from "vue-router";
import { ElLoading, ElMessage } from "element-plus";
import { key } from "../store";
import { isMobileOrPc, getQueryStringByName } from "../utils/utils";
import { UserInfo, NavListItem } from "../types/index";

const RegisterAndLogin = defineAsyncComponent(
  () => import("./RegisterAndLogin.vue")
);

const store = useStore(key);
const router = useRouter();

//#region 定义响应式数据
const visible = ref(false);
const handleFlag = ref("");
const title = ref("首页");
const list = ref<Array<NavListItem>>([
  {
    index: "1",
    path: "/",
    name: "首页",
  },
  {
    index: "2",
    path: "/articles",
    name: "文章",
  },
  {
    index: "3",
    path: "/archive",
    name: "归档",
  },
  {
    index: "4",
    path: "/project",
    name: "项目",
  },
  {
    index: "5",
    path: "/timeline",
    name: "历程",
  },
  {
    index: "6",
    path: "/message",
    name: "留言",
  },
  {
    index: "7",
    path: "/about",
    name: "关于",
  },
]);
const activeIndex = ref("0");
const enterSlideUp = ref(false);
const leaveSlideDown = ref(false);
const isShow = ref(false);
const isMobile = ref(isMobileOrPc());
//#endregion

const userInfo = computed((): UserInfo => {
  let userInfo: UserInfo = {
    _id: "",
    name: "",
    avatar: "",
  };
  if (window.sessionStorage.userInfo) {
    userInfo = JSON.parse(window.sessionStorage.userInfo);
    (this as any).$store.commit("SAVE_USER", {
      userInfo,
    });
  }
  if ((this as any).$store.state.user.userInfo) {
    userInfo = (this as any).$store.state.user.userInfo;
  }
  return userInfo;
});

watch($route, (val: any, oldVal: any) => {
  routeChange(val, oldVal);
});

//#region method
const routeChange = (val: any, oldVal: any) => {
  for (let i = 0; i < list.value.length; i++) {
    const l: NavListItem = list.value[i];
    if (l.path === val.path) {
      activeIndex.value = i + 1 + "";
      title.value = l.name;
      break;
    }
  }
};

const handleSelect = (val: string, oldVal: string): void => {
  activeIndex.value = val;
};

const handleOk = (value: boolean): void => {
  visible.value = value;
};

const handleCancel = (value: boolean): void => {
  visible.value = value;
};

const handleClick = (value: string): void => {
  handleFlag.value = value;
  visible.value = true;
};

const handleLogout = (): void => {
  window.sessionStorage.userInfo = "";
  store.commit("SAVE_USER", {
    userInfo: {
      _id: "",
      name: "",
      avatar: "",
    },
  });
};

const handleClickMenu = (route?: string): void => {
  isShow.value = false;
  if (route === "/login") {
    handleFlag.value = "login";
    visible.value = true;
  }
  if (route === "/register") {
    handleFlag.value = "register";
    visible.value = true;
  }
  if (route === "/logout") {
    handleLogout();
  }
};

const handleMenu = (): void => {
  isShow.value = true;
  enterSlideUp.value = true;
};

const getUser = async (code: string): Promise<void> => {
  const loading: any = ElLoading.service({
    lock: true,
    text: "Loading",
    spinner: "el-icon-loading",
    background: "rgba(255, 255, 255, 0.7)",
  });
  const data: UserInfo = await service.post(
    urls.getUser,
    { code },
    { withCredentials: true }
  );
  loading.close();

  const userInfo: UserInfo = {
    _id: data._id,
    name: data.name,
    avatar: data.avatar,
  };
  store.commit("SAVE_USER", {
    userInfo,
  });
  window.sessionStorage.userInfo = JSON.stringify(userInfo);
  ElMessage({
    message: "操作成功",
    type: "success",
  });
  let preventHistory = JSON.parse(window.sessionStorage.preventHistory);
  if (preventHistory) {
    router.push({
      path: preventHistory.name,
      query: preventHistory.query,
    });
  }
};

const handleHideMenu = (): void => {
  enterSlideUp.value = false;
  leaveSlideDown.value = true;
  setTimeout(() => {
    leaveSlideDown.value = false;
    isShow.value = false;
  }, 300);
};

//#endregion
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
.nav-mobile {
  display: flex;
  line-height: 60px;
  .nav-mobile-logo {
    flex: 1;
    margin-top: 5px;
    margin-left: 10px;
  }
  .title {
    flex: 3;
    font-size: 24px;
  }
  .menu {
    flex: 1;
    font-size: 34px;
    color: #409eff;
  }
}
.nav-mobile-content {
  font-size: 0.3rem;
  height: 7.3rem;
  width: 100%;
  background-color: #fff;
  .list {
    .item {
      line-height: 0.8rem;
      color: #303133;
      border-bottom: 1px solid #eee;
      a {
        display: block;
        width: 100%;
        color: #409eff;
        text-decoration-line: none;
      }
    }
  }
}
.nav {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1000;
  width: 100%;
  border-bottom: 1px solid #eee;
  background-color: #fff;
  .nav-content {
    width: 1200px;
    margin: 0 auto;
  }
  .logo {
    height: 50px;
    margin: 0;
    border-radius: 50%;
    margin-top: 5px;
  }
  .el-menu.el-menu--horizontal {
    border-bottom: none;
  }
  .el-menu--horizontal > .el-menu-item {
    cursor: pointer;
    color: #333;
  }
  .nav-right {
    position: relative;
    padding-top: 15px;
    text-align: right;
    .el-dropdown {
      cursor: pointer;
      padding-right: 60px;
    }
    .user-img {
      position: absolute;
      top: -15px;
      right: 0;
      width: 50px;
      border-radius: 50%;
    }
  }
}

.enter-slideUp,
.leave-slideDown {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1010;
}

.enter-slideUp {
  overflow: auto;
  visibility: visible;
  z-index: 1001;
  animation: slideUp 0.3s forwards;
}
.leave-slideDown {
  visibility: visible;
  z-index: 1001;
  animation: slideDown 0.3s forwards;
}
@keyframes slideUp {
  from {
    transform: translate3d(0, 100%, 0);
    opacity: 0.1;
  }
  to {
    transform: translate3d(0, 0, 0);
    opacity: 1;
  }
}
@keyframes slideDown {
  from {
    transform: translate3d(0, 0, 0);
  }
  to {
    transform: translate3d(0, 100%, 0);
    opacity: 0;
  }
}
.mask {
  position: fixed;
  z-index: 100;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #000;
  opacity: 0.5;
}
.mask-fade-out {
  animation: maskFadeOut 0.4s forwards;
}
@keyframes maskFadeOut {
  from {
    opacity: 0.5;
  }
  to {
    opacity: 0;
  }
}
</style>
