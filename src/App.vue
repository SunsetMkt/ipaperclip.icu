<template>
  <n-config-provider :locale="zhCN" :theme="darkTheme">
    <n-loading-bar-provider>
      <div style="text-align: center; padding-top: 15px">
        <n-h1>
          <a style="color: #ffffffe6; text-decoration: none" href="/">
            iPaperClipICU
          </a>
        </n-h1>
      </div>
      <n-grid :cols="36" item-responsive style="padding-top: 20px">
        <n-gi span="1 1330:3" />
        <n-gi span="34 1330:30">
          <router-view />
          <div style="padding-top: 15px">
            <READMECard />
          </div>
        </n-gi>
        <n-gi span="1 1330:3" />
      </n-grid>
    </n-loading-bar-provider>
    <n-global-style />
  </n-config-provider>
</template>

<script>
import { defineComponent, ref } from "vue";
import READMECard from "@/components/READMECard";
import { NH1, NGi, NGrid } from "naive-ui";
import { NGlobalStyle, NConfigProvider, NLoadingBarProvider } from "naive-ui";
import { darkTheme, zhCN } from "naive-ui";

const mailLoginShow = ref(true);
export default defineComponent({
  components: {
    READMECard,
    NH1,
    NGi,
    NGrid,
    NGlobalStyle,
    NConfigProvider,
    NLoadingBarProvider,
  },
  setup() {
    return {
      zhCN,
      darkTheme,
      mailLoginShow,
    };
  },
});
</script>
