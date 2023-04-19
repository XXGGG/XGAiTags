<template>
  <div id="app">
    <n-config-provider :theme="theme">
      <n-global-style />
      <n-message-provider>
        <MSG ref="msgg" />
      </n-message-provider>
      <!-- 顶部导航栏 -->
      <n-card class="card_class" hoverable>
        <div class="Top">
          <div>XG AiTags</div>
          <n-space>
            <n-button @click="show_Tip = true" v-if="!show_Tip">Tip</n-button>
            <n-button @click="theme = darkTheme" v-if="theme == null">
              <n-icon size="20"><SunnySharp /></n-icon>
            </n-button>
            <n-button @click="theme = null" v-else>
              <n-icon size="20"><MoonOutline /></n-icon>
            </n-button>
            <a href="https://github.com/XXGGG/XGAiTags" target="_blank">
              <n-button>
                <n-icon size="20">
                  <LogoGithub />
                </n-icon>
              </n-button>
            </a>
          </n-space>
        </div>
      </n-card>
      <n-card
        closable
        hoverable
        @close="Close_hint()"
        v-show="show_Tip"
        class="card_class"
        title="Tip">
        <p>
          🚀 优秀的关键词 🟰 画质要求 ➕ 风格类型 ➕ 人物/物品描述 ➕ 场景描述 ➕
          细节补充
        </p>
        <p>🖼️ 优秀的AI绘画 🟰 风格相应的模型 ➕ 优秀的关键词 ➕ 反关键词</p>
        <p>
          ⚠️ 注意：选择【风格】时，最好有对应风格的模型（下载模型的地址：
          <a href="https://civitai.com/">Civitai（需要翻墙）</a>
          |
          <a href="https://models.paomiantv.cn/models/">炼丹阁</a>
          ）
        </p>
      </n-card>

      <!-- 展示框 -->
      <n-card class="card_class" hoverable>
        <div class="remote_control">
          <div class="remote_control_l">
            <n-switch
              size="large"
              v-model:value="showWeight"
              class="Button_Size">
              <template #checked>权重</template>
              <template #unchecked>权重</template>
            </n-switch>
            <n-switch size="large" v-model:value="showR18" class="Button_Size">
              <template #checked-icon>😈</template>
              <template #unchecked-icon>🔞</template>
              <template #checked>R18</template>
              <template #unchecked>R18</template>
            </n-switch>
            <!-- <n-switch
              size="large"
              v-model:value="togglebracket"
              class="Button_Size"
            >
              <template #checked> { } </template>
              <template #unchecked> ( ) </template>
            </n-switch> -->
          </div>
          <div class="remote_control_r">
            <n-button
              type="success"
              secondary
              class="Button_Size"
              @click="copy_all">
              复制
            </n-button>
            <n-button
              type="error"
              secondary
              class="Button_Size"
              @click="close_all">
              清空
            </n-button>
          </div>
        </div>

        <div class="Prompt_class">
          <div class="prompt_show_box">{{ prompt }}</div>
          <div class="prompt_show_box">
            <TransitionGroup name="list">
              <div
                v-for="(tag, index) in tags"
                :key="tag"
                class="tag_class"
                draggable="true"
                @dragstart.self="dragstart($event, index)"
                @dragenter="dragenter($event, index)"
                @dragend="dragend($event, index)">
                <n-button-group size="small">
                  <n-button
                    v-if="showWeight"
                    size="small"
                    :type="tag.R18 == 'false' ? 'primary' : 'error'"
                    ghost
                    @click="addKH(tag)">
                    <n-icon size="20" color="#5ae6ae">
                      <Add />
                    </n-icon>
                  </n-button>
                  <n-button
                    v-if="showWeight"
                    size="small"
                    :type="tag.R18 == 'false' ? 'primary' : 'error'"
                    ghost
                    @click="subKH(tag)">
                    <n-icon size="20" color="#5ae6ae">
                      <Remove />
                    </n-icon>
                  </n-button>
                  <n-button
                    :type="tag.R18 == 'false' ? 'primary' : 'error'"
                    ghost>
                    {{ tag.en }}&nbsp
                    <n-gradient-text type="info">
                      {{ tag.zh }}&nbsp
                    </n-gradient-text>
                  </n-button>
                  <n-button
                    :type="tag.R18 == 'false' ? 'primary' : 'error'"
                    ghost
                    @click="closeTag(tag)">
                    <n-icon size="20" color="#c83838">
                      <Close />
                    </n-icon>
                  </n-button>
                </n-button-group>
              </div>
            </TransitionGroup>
          </div>
        </div>
      </n-card>

      <!-- 选项框 -->
      <n-card class="card_class" hoverable>
        <n-tabs type="line" animated>
          <n-tab-pane
            v-for="(SB, index) in StatuteBook"
            :name="SB.category"
            :tab="SB.category">
            <div v-for="t in SB.types" v-show="SB.category != 'R18' || showR18">
              <h3>{{ t.name }}</h3>
              <n-button
                class="tag_button"
                v-for="(tag, index) in t.tags"
                @click="to_prompt(tag)"
                :type="tag.R18 == 'false' ? 'success' : 'error'"
                :dashed="selected(tag)"
                :ghost="selected(tag)"
                :secondary="!selected(tag)"
                v-show="tag.R18 == 'false' || showR18">
                {{ tag.en }}&nbsp
                <n-gradient-text type="info">{{ tag.zh }}</n-gradient-text>
              </n-button>
            </div>
          </n-tab-pane>
        </n-tabs>
      </n-card>
    </n-config-provider>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import StatuteBook2 from "./assets/StatuteBook.json";
import { useClipboard, useStorage } from "@vueuse/core";
import {
  Add,
  Remove,
  Close,
  LogoGithub,
  SunnySharp,
  MoonOutline,
} from "@vicons/ionicons5";
import {
  NGlobalStyle,
  NConfigProvider,
  darkTheme,
  NButton,
  NCard,
  NTag,
  NTabs,
  NTabPane,
  NGradientText,
  NButtonGroup,
  NSwitch,
  NIcon,
  NSpace,
  NMessageProvider,
} from "naive-ui";
import MSG from "./components/msg.vue";
import type { GlobalTheme } from "naive-ui";

const theme = ref<GlobalTheme | null>(darkTheme);

// 以下vueuse用于复制
const source = ref("");
const { copy } = useClipboard({ source });

let prompt = ref("");
let tags: any = ref([]);

const StatuteBook = StatuteBook2;

function to_prompt(tag: any) {
  // console.log(tag);
  let exist = tags.value.includes(tag);
  if (exist) {
    let index = tags.value.indexOf(tag);
    tags.value.splice(index, 1);
  } else {
    tags.value.push(tag);
  }
  JoinWord();
}

//无论做何操作，最后都需要重新连接所有关键词
function JoinWord() {
  // console.log(tags)
  let tags_en: any = [];
  tags.value.forEach(function (x: any) {
    tags_en.push(x.en);
  });
  // console.log(tags_en)
  prompt.value = tags_en.join(",");
}

//删除对应标签
function closeTag(tag: any) {
  let index = tags.value.indexOf(tag);
  // console.log(tag)
  //去掉左右两边的括号！
  if (tag.en[0] == "(" || tag.en[0] == "[" || tag.en[0] == "{") {
    tag.en = tag.en.slice(1, -1);
    // tags.value.splice(index, index == 0 ? "1" : index, tag);
    closeTag(tag);
  } else {
    tags.value.splice(index, 1);
    JoinWord();
  }
}

//清空所有标签
function close_all() {
  tags.value = [];
  JoinWord();
}

const msgg: any = ref(null);
//复制标签！
function copy_all() {
  copy(prompt.value);
  msgg.value.show();
}

//判断是否选择了这个tag
function selected(tag: any) {
  let exist = tags.value.includes(tag);
  if (exist) {
    return false;
  } else {
    return true;
  }
}
//是否显示权重
// let showWeight = ref(false);
const showWeight = useStorage("showWeight", false);
//是否显示18禁
// let showR18 = ref(false);
const showR18 = useStorage("showR18", false);
//切换括号
let togglebracket = ref(false);

//加重权重
function addKH(tag: any) {
  let index = tags.value.indexOf(tag);
  if (tag.en[0] == "[") {
    tag.en = tag.en.slice(1, -1);
  } else {
    tag.en = "(" + tag.en + ")";
  }
  // tags.value.splice(index, index == "0" ? "1" : index, tag);
  JoinWord();
}
//减少权重
function subKH(tag: any) {
  let index = tags.value.indexOf(tag);

  if (tag.en[0] == "(") {
    tag.en = tag.en.slice(1, -1);
  } else {
    tag.en = "[" + tag.en + "]";
  }
  // tags.value.splice(index, index == "0" ? "1" : index, tag);
  JoinWord();
}

// let show_Tip = ref(true);
const show_Tip = useStorage("show_Tip", true);
//关闭提示
function Close_hint() {
  show_Tip.value = false;
}

///拖拽
let in_index: any = null;
let out_index: any = null;
let in_event: any = null;

let cloneObj: any = null; //克隆对象
let left: number;
let top: number;
let layerX: number; //鼠标点击物体里面的方位
let layerY: number;

let moveing = false;
//1.开始拖拽触发器
function dragstart(ev: any, index: any) {
  // console.log("拖拽开始！");
  //先把原来的“幽灵图片”隐藏
  var img = new Image();
  img.src = "";
  ev.dataTransfer.setDragImage(img, 0, 0);

  //把当前 元素 给到大局
  in_event = ev;

  //把当前的index 给到大局
  in_index = index;

  //把拖拽时，原来的底图也暂时隐藏掉
  in_event.target.style.opacity = "0";

  //这里先拿到鼠标点进 元素里的坐标
  layerX = ev.layerX; //鼠标点击物体里面的方位
  layerY = ev.layerY;

  //用鼠标在浏览器窗口的坐标 减去 鼠标在元素里的坐标 = 元素偏移后对应鼠标当前位置在浏览器上的坐标
  left = ev.clientX - layerX;
  top = ev.clientY - layerY;

  //克隆节点！
  let el = ev.target;
  cloneObj = el.cloneNode(true);
  cloneObj.style =
    "position:absolute;left:0;top:0;z-index:9999999;pointer-events:none;transform:translate3d( " +
    left +
    "px ," +
    top +
    "px,0);";
  //生成这个节点出来，节点的偏移坐标，已经在上面 当前鼠标的位置 计算好了！
  document.body.appendChild(cloneObj);
}
//监听全局拖拽事件，同样计算好偏移。做到跟随鼠标！！
document.addEventListener("dragover", function (ev) {
  left = ev.clientX - layerX;
  top = ev.clientY - layerY;
  if (cloneObj) {
    cloneObj.style.transform = "translate3d( " + left + "px ," + top + "px,0)";
  }
});
//监听，当拖拽结束后，就把生成的那个节点给删掉！！
document.addEventListener("dragend", function (ev) {
  document.body.removeChild(cloneObj);
  cloneObj = null;
});

setInterval(() => {
  moveing = false;
}, 400);

//2.拖拽进入👇
function dragenter(ev: any, index: any) {
  if (!moveing) {
    moveing = true;
    out_index = index;
    // console.log(in_index);
    // console.log(out_index);
    if (in_index > out_index) {
      tags.value.splice(out_index, 0, tags.value[in_index]);
      tags.value.splice(in_index + 1, 1);
    } else if (in_index < out_index) {
      tags.value.splice(out_index + 1, 0, tags.value[in_index]);
      tags.value.splice(in_index, 1);
    }
    in_index = index;
    JoinWord();
  }
}
let dragend = (event: any, index: any) => {
  //回复原节点元素的 显示状态！
  in_event.target.style.opacity = "1";
};
</script>
<style style="scss" scoped>
.Top {
  display: flex !important;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}
.card_class {
  width: calc(100% - 20px);
  margin: 10px auto;
}
.Prompt_class {
  display: flex;
  flex-direction: column;
}

.prompt_show_box {
  width: 100%;
  /* background-color: rgba(164, 164, 164, 0.1); */
  min-height: 100px;
  padding: 5px 10px;
  box-sizing: border-box;
  margin: 5px 0;
  font-size: 20px;
}

.Button_Size {
  /* font-size: 20px; */
  margin: 0 10px;
}
.tag_class {
  margin: 0 10px 10px 0;
  user-select: none;
  float: left;
}
.tag_button {
  margin: 0 10px 10px 0;
}

.remote_control {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.remote_control_l {
  display: flex;
  align-items: center;
}
.remote_control_r {
  display: flex;
  align-content: center;
  color: #5ae6ae;
}

.list-move {
  transition: all 0.2s;
}
</style>
