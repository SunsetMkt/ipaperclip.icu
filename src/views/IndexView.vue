<template>
  <n-card hoverable>
    <n-grid :cols="4" item-responsive>
      <n-gi span="4 425:2 705:1">
        <n-input-group>
          <n-input v-model:value="searchValue" placeholder="搜索" clearable />
          <n-button type="primary" ghost @click="searchButton">搜索</n-button>
        </n-input-group>
      </n-gi>
      <n-gi span="0 425:2 705:3" />
    </n-grid>
  </n-card>
  <div style="padding-top: 15px">
    <n-card hoverable>
      <!-- 菜单 -->
      <CMenu />
      <n-divider />
      <!-- 文件夹 -->
      <div v-if="showFilesMenu == true">
        <FilesMenu :data="FilesMenu_data" />
      </div>
      <!-- Null -->
      <n-empty v-if="showEmpty" description="请在「菜单」中选择" />
      <n-empty v-if="showNullEmpty" description="找不到和查询相匹配的结果" />
      <!-- 文件 -->
      <div v-if="showShowFile" style="margin-top: 15px">
        <ShowFile :data="ShowFile_data" />
      </div>
    </n-card>
  </div>
</template>

<script>
import { h, ref, defineComponent } from "vue";
import router from "@/router";
import {
  NGi,
  NGrid,
  NCard,
  NEmpty,
  NInput,
  NSpace,
  NButton,
  NDivider,
  NDropdown,
  NInputGroup,
} from "naive-ui";
import ShowFile from "@/components/ShowFile";
import FilesMenu from "@/components/FilesMenu";
import data from "@/assets/data.json";
import { getFileInfo } from "@/assets/box.js";

/**
 * 获取data
 */
const getData = (data, name) => {
  if (data[name] != void 0) {
    return data[name];
  } else {
    // 没有找到
    showFilesMenu.value = false;
    showEmpty.value = true;
    showShowFile.value = false;
    return null;
  }
};

/**
 * /
 * /files
 * /files/tag
 * /files/tag/file
 * /files/file
 */

const init = () => {
  const path = location.pathname;

  showFilesMenu.value = false;
  showEmpty.value = false;
  showShowFile.value = false;

  if (path == "/") {
    // Path: /
    showEmpty.value = true;
  } else {
    const filesName = decodeURIComponent(path.split("/")[1]);
    const filesData = getData(data.data, filesName);
    if (filesData == null) return;

    if (filesData.pages == void 0) {
      // 有Tag
      var tagName = decodeURIComponent(path.split("/")[2]);
      if (tagName == "undefined") {
        for (const i in data.menuData) {
          if (filesName == data.menuData[i][0]) {
            tagName = data.menuData[i][1][0];
            break;
          }
        }
        router.push(`/${filesName}/${tagName}`);
      }

      const tagData = getData(filesData, tagName);
      if (tagData == null) return;
      const fileName = decodeURIComponent(path.split("/")[3]);

      if (fileName == "undefined" || fileName == "") {
        // Path: /files/tag
        FilesMenu_data.value = {
          hrefHead: `/${filesName}/${tagName}`,
          data: tagData,
        };
        showFilesMenu.value = true;
      } else {
        // Path: /files/tag/file
        var fileNameW = undefined;
        if (tagData.pages) {
          for (const i in tagData.data) {
            for (const ii in tagData.data[i]) {
              if (fileName == tagData.data[i][ii].substr(0, fileName.length)) {
                fileNameW = tagData.data[i][ii];
              }
            }
          }
        } else {
          for (const i in tagData.data) {
            if (fileName == tagData.data[i].substr(0, fileName.length)) {
              fileNameW = tagData.data[i];
            }
          }
        }
        if (fileNameW == undefined) {
          showFilesMenu.value = false;
          showEmpty.value = true;
          showShowFile.value = false;
          return;
        }

        ShowFile_data.value = {
          type: getFileInfo(fileNameW).type,
          name: fileName,
          url: `https://file.hsyhx.top/video/${filesName}/${tagName}/${fileNameW}`,
        };
        showShowFile.value = true;
      }
    } else {
      const fileName = decodeURIComponent(path.split("/")[2]);

      if (fileName == "undefined" || fileName == "") {
        // Path: /files
        FilesMenu_data.value = {
          hrefHead: `/${filesName}`,
          data: filesData,
        };
        showFilesMenu.value = true;
      } else {
        // Path: /files/file
        var fileNameM = undefined;
        if (filesData.pages) {
          for (const i in filesData.data) {
            for (const ii in filesData.data[i]) {
              if (
                fileName == filesData.data[i][ii].substr(0, fileName.length)
              ) {
                fileNameM = filesData.data[i][ii];
              }
            }
          }
        } else {
          for (const i in filesData.data) {
            if (fileName == filesData.data[i].substr(0, fileName.length)) {
              fileNameM = filesData.data[i];
            }
          }
        }
        if (fileNameM == undefined) {
          showFilesMenu.value = false;
          showEmpty.value = true;
          showShowFile.value = false;
          return;
        }

        ShowFile_data.value = {
          type: getFileInfo(fileNameM).type,
          name: fileName,
          url: `https://file.hsyhx.top/video/${filesName}/${fileNameM}`,
        };
        showShowFile.value = true;
      }
    }
  }
};

const getCMenu = () => {
  const filesName = decodeURIComponent(location.pathname.split("/")[1]);
  const getButton = (name, at, click) => {
    let type = "";
    if (at) type = "primary";
    else type = "default";

    return h(
      NButton,
      {
        text: true,
        size: "large",
        type,
        "on-click": () => {
          if (click) location.href = `/${name}`;
        },
      },
      {
        default: () => name,
      }
    );
  };

  let MenuList = [];
  for (const i in data.menuData) {
    const MenuData = data.menuData[i];
    let at;
    if (filesName == MenuData[0]) at = true;
    else at = false;

    if (MenuData.length == 2) {
      // 有Tag
      let options = [];
      for (const ii in MenuData[1]) {
        options.push({
          label: MenuData[1][ii],
          key: `/${MenuData[0]}/${MenuData[1][ii]}`,
        });
      }

      MenuList.push(
        h(
          NDropdown,
          {
            trigger: "hover",
            options: options,
            "on-select": (key) => {
              location.href = key;
            },
          },
          {
            default: () => getButton(MenuData[0], at, false),
          }
        )
      );
    } else {
      // 没有Tag
      MenuList.push(getButton(MenuData[0], at, true));
    }
  }

  return h(
    NSpace,
    {
      justify: "center",
      size: 30,
    },
    {
      default: () => MenuList,
    }
  );
};

const searchValue = ref("");
const CMenu = getCMenu();
// Show
const showEmpty = ref(false);
const showNullEmpty = ref(false);
const showShowFile = ref(false);
const showFilesMenu = ref(false);
// Data
const FilesMenu_data = ref({
  hrefHead: "/test",
  search: false,
  data: [
    {
      name: "test",
    },
  ],
});
// const FilesMenu_data = ref({
//   search: true,
//   data: [
//     {
//       name: "test",
//       hrefHead: "/test",
//       tag: "[XXX]",
//     },
//   ],
// });
const ShowFile_data = ref({
  type: "audio",
  name: "test audio",
  url: "https://file.hsyhx.top/video/test.mp3",
});
export default defineComponent({
  components: {
    CMenu,
    ShowFile,
    FilesMenu,
    NGi,
    NGrid,
    NCard,
    NEmpty,
    NInput,
    NButton,
    NDivider,
    NInputGroup,
  },
  setup() {
    init();
    return {
      // Show
      showEmpty,
      showShowFile,
      showNullEmpty,
      showFilesMenu,
      // Data
      ShowFile_data,
      FilesMenu_data,
      // 搜索
      searchValue,
      searchButton: async (e) => {
        e.preventDefault();

        if (searchValue.value == "") return;

        showEmpty.value = false;
        showShowFile.value = false;
        showNullEmpty.value = false;
        showFilesMenu.value = false;

        let searchData = {
          search: true,
          data: [],
        };

        const KeyWord = String(searchValue.value).toLocaleLowerCase();
        for (const i in data.searchData) {
          if (i.toLocaleLowerCase().indexOf(KeyWord) != -1) {
            if (data.searchData[i].length == 2) {
              // 有Tag
              searchData.data.push({
                name: i,
                hrefHead: `/${data.searchData[i][0]}/${data.searchData[i][1]}`,
                tag: `[${data.searchData[i][0]}/${data.searchData[i][1]}]`,
              });
            } else {
              // 没有Tag
              searchData.data.push({
                name: i,
                hrefHead: `/${data.searchData[i][0]}`,
                tag: `[${data.searchData[i][0]}]`,
              });
            }
          }
        }

        if (searchData.data.length == 0) {
          showNullEmpty.value = true;
        } else {
          FilesMenu_data.value = searchData;
          showFilesMenu.value = true;
        }
      },
    };
  },
});
</script>
