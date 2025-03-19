<script lang="ts">
import { ChatUIKit } from "./ChatUIKit";
import {
  APPKEY,
  API_URL,
  URL,
  CHAT_STORE,
  getInsideGroupAvatarUrl
} from "@/const/index";
import websdk from "easemob-websdk/uniApp/Easemob-chat";
import { EasemobChatStatic } from "easemob-websdk/Easemob-chat";
import GroupNewAvatar from "./ChatUIKit/assets/groupNew.png";
import { autorun, runInAction } from "mobx";

const chat = new (websdk as unknown as EasemobChatStatic).connection({
  appKey: APPKEY,
  isHttpDNS: true,
  url: URL,
  apiUrl: API_URL,
  delivery: true
});

// websdk.logger.disableAll();

ChatUIKit.init({
  chat,
  config: {
    theme: {
      avatarShape: "circle"
    },
    isDebug: false
  }
});

ChatUIKit.hideFeature(["useUserInfo"]);

ChatUIKit.getChatConn().addEventHandler("chat", {
  onMessage: (messages) => {
    messages.forEach((message) => {
      const { ease_chat_uikit_user_info } = message.ext || {};
      const { nickname, avatarURL } = ease_chat_uikit_user_info || {};
      if (!ChatUIKit.appUserStore.getUserInfoFromStore(message.from).nickname) {
        ChatUIKit.appUserStore.setUserInfo(message.from, {
          nickname: nickname,
          avatarurl: avatarURL
        });
      }
    });
  }
});

// 手动设置用户属性
// ChatUIKit.appUserStore.setUserInfo("0c1bdd28c7", {
//   nickname: "张三",
//   avatarurl: "https://p9-passport.byteacctimg.com/img/user-avatar/6d239ae53c4aded5fadd95cda5fc6759~40x40.awebp"
// });

uni.$UIKit = ChatUIKit;

// 监听群组变化获取群组头像
autorun(() => {
  const groupIds = ChatUIKit.groupStore.joinedGroupList
    .filter((group) => {
      // 过滤掉已经有头像的群组
      return !ChatUIKit.groupStore.isHasGroupAvatar(group.groupId);
    })
    .map((group) => {
      // 设置头像空头像, 避免重复请求
      ChatUIKit.groupStore.setGroupAvatar(group.groupId, "");
      return group.groupId;
    });

  if (groupIds.length > 0) {
    getGroupAvatarUrl(groupIds);
  }
});

// 获取群组头像
const getGroupAvatarUrl = async (groupIds: string[]) => {
  for (let groupId of groupIds) {
    try {
      const res = await ChatUIKit.groupStore.getGroupInfo(groupId);
      runInAction(() => {
        // 设置群组头像
        //@ts-ignore
        ChatUIKit.groupStore.setGroupAvatar(
          groupId,
          res.data[0].avatar || GroupNewAvatar
        );
      });
    } catch (error) {
      console.error("Failed to fetch group avatar:", groupId, error);
    }
  }
};

const autoLogin = async () => {
  try {
    uni.showLoading({
      title: "加载中"
    });
    const res: any = await uni.request({
      url: "https://a1-appserver.easemob.com/inside/app/user/special/login",
      header: {
        "content-type": "application/json"
      },
      method: "POST"
    });
    // 如果存在缓存，直接登录
    if (res.data) {
      const { chatUserName, token } = res.data;
      await uni.$UIKit.chatStore.login({
        user: chatUserName,
        accessToken: token
      });
      uni.hideLoading();
      // 跳转会话列表页面
      uni.reLaunch({
        url: "/ChatUIKit/modules/Conversation/index",
        success: () => {
          // #ifdef APP-PLUS
          plus.navigator.closeSplashscreen();
          // #endif
        }
      });
    }
  } catch (error) {}
};

export default {
  onLaunch: function () {
    console.log("App Launch");
    autoLogin();
  },
  onShow: function () {
    console.log("App Show");
    ChatUIKit.onShow();
  },
  onHide: function () {
    console.log("App Hide");
  }
};
</script>

<style>
@import url("./common.scss");
</style>
