<template>
  <view class="search-list-wrap">
    <NavBar @onLeftTap="cancelSearch">
      <template v-slot:left>
        <view class="input-wrap">
          <SearchInput
            ref="searchRef"
            :placeholder="t('searchContact')"
            @input="onInput"
            @cancel="cancelSearch"
          />
        </view>
      </template>
    </NavBar>

    <view class="search-content" v-if="searchList.length">
      <checkbox-group @change="checkboxChange">
        <label
          class="label"
          v-show="searchList.includes(item.userId)"
          v-for="item in contactList"
        >
          <checkbox
            class="checkbox"
            backgroundColor="#f9fafa"
            borderColor="#ACB4B9"
            activeBackgroundColor="#009DFF"
            activeBorderColor="#009DFF"
            style="transform: scale(0.8)"
            iconColor="#fff"
            :value="item.userId"
            :checked="props.checkedList.includes(item.userId)"
          />
          <UserItem
            :key="item.conversationId"
            class="search-item"
            :user="{ userId: item.userId }"
          />
        </label>
      </checkbox-group>
    </view>
    <Empty v-else />
  </view>
</template>

<script setup lang="ts">
import NavBar from "../../components/NavBar/index.vue";
import SearchInput from "../../components/SearchInput/index.vue";
import UserItem from "../ContactList/components/UserItem/index.vue";
import Empty from "../../components/Empty/index.vue";
import { ChatUIKit } from "../../index";
import { t } from "../../locales";
import { ref, computed } from "../../composition-api-adaptor";
import { autorun } from "mobx";
import { Chat } from "../../../../ChatUIKit/sdk";
import { onUnmounted } from "../../composition-api-adaptor";
import { deepClone } from "../../utils";
interface Props {
  checkedList: string[];
}

const props = defineProps<Props>();

const emits = defineEmits(["checkboxChange", "cancel"]);

const contactList = ref<Chat.ContactItem[]>([]);

const searchRef = ref(null);

const searchValue = ref("");

const onInput = (value: string) => {
  searchValue.value = value;
};

const checkboxChange = (e) => {
  const values = e.detail.value;
  emits("checkboxChange", values);
};

const unwatchContactList = autorun(() => {
  contactList.value = deepClone(ChatUIKit.contactStore.contacts);
});

const searchList = computed(() => {
  if (!searchValue.value) {
    return [];
  }
  return contactList.value
    .filter((item) => {
      return ChatUIKit.appUserStore
        .getUserInfoFromStore(item.userId)
        .name.includes(searchValue.value);
    })
    .map((filterItem) => {
      return filterItem.userId;
    });
});

const cancelSearch = () => {
  emits("cancel");
};

onUnmounted(() => {
  unwatchContactList();
});
</script>

<style lang="scss" scoped>
.label {
  display: flex;
  width: 100%;
  align-items: center;
}

.search-content {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 0 16px;
  overflow-y: scroll;
}

.search-item {
  flex: 1;
}

.input-wrap {
  /*  #ifndef MP-WEIXIN  */
  width: calc(100vw - 50px);
  /*  #endif  */
  /*  #ifdef MP-WEIXIN  */
  width: calc(100vw - 150px);
  /*  #endif  */
}

.search-list-wrap {
  height: calc(100% - var(--status-bar-height));
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding-top: 5px;
}
</style>
