<template>
  <div id="app">
    <n-config-provider :theme="theme">
      <n-global-style />
      <n-message-provider>
        <MSG ref="msgg" />
      </n-message-provider>
      <!-- 展示框 -->
      <n-card class="card_class" hoverable>
        <div class="remote_control">
          <div class="remote_control_l">
            <n-switch size="large" v-model:value="showWeight" class="Button_Size" :round="false">
              <template #checked-icon>
                ❗
              </template>
              <template #unchecked-icon>
                ❕
              </template>
              <template #checked>权重</template>
              <template #unchecked>权重</template>
            </n-switch>
            <n-switch size="large" v-model:value="showClose" class="Button_Size" :round="false">
              <template #checked-icon>
                <Close />
              </template>
              <template #unchecked-icon>
                <Close />
              </template>
              <template #checked>删除</template>
              <template #unchecked>删除</template>
            </n-switch>
            <n-switch size="large" v-model:value="showR18" class="Button_Size">
              <template #checked-icon>😈</template>
              <template #unchecked-icon>🔞</template>
              <template #checked>R18</template>
              <template #unchecked>R18</template>
            </n-switch>
            <n-switch size="large" v-model:value="OpenCopy" class="Button_Size">
              <template #checked-icon></template>
              <template #unchecked-icon></template>
              <template #checked>单击复制</template>
              <template #unchecked>单击复制</template>
            </n-switch>

          </div>
          <div class="remote_control_r">
            <n-space>
              <n-icon size="20" class="XGicon_tr" @click="theme = darkTheme" v-if="theme == null">
                <SunnySharp />
              </n-icon>
              <n-icon size="20" class="XGicon_tr" @click="theme = null" v-else>
                <MoonOutline />
              </n-icon>
              <n-icon size="20" class="XGicon_tr" @click="goUrl('https://github.com/XXGGG/XGAiTags')" title="Github">
                <LogoGithub />
              </n-icon>
              <n-icon size="20" class="XGicon_tr" @click="goUrl('https://civitai.com/user/XXGGG')" title="C站">
                <LogoWebComponent />
              </n-icon>
              <n-icon size="20" class="XGicon_tr" @click="goUrl('https://space.bilibili.com/5276030')" title="B站">
                B
              </n-icon>
            </n-space>
          </div>
        </div>

        <div class="Prompt_class">
          <div class="prompt_show_box">{{ prompt }}</div>
          <div class="prompt_show_box">
            <TransitionGroup name="list">
              <div v-for="(tag, index) in tags" :key="tag" class="tag_class" draggable="true"
                @dragstart.self="dragstart($event, index)" @dragenter="dragenter($event, index)"
                @dragend="dragend($event, index)">
                <n-button-group size="small">
                  <n-button v-if="showClose" :type="tag.R18 == 'false' ? 'primary' : 'error'" strong secondary
                    @click="closeTag(tag)">
                    <n-icon size="20" color="#c83838">
                      <Close />
                    </n-icon>
                  </n-button>
                  <n-button v-if="showWeight" size="small" :type="tag.R18 == 'false' ? 'primary' : 'error'" strong
                    secondary @click="addKH(tag)">
                    <n-icon size="20" color="#5ae6ae">
                      <Add />
                    </n-icon>
                  </n-button>
                  <n-button v-if="showWeight" size="small" :type="tag.R18 == 'false' ? 'primary' : 'error'" strong
                    secondary @click="subKH(tag)">
                    <n-icon size="20" color="#5ae6ae">
                      <Remove />
                    </n-icon>
                  </n-button>
                  <n-button strong secondary :type="tag.R18 == 'false' ? 'primary' : 'error'"
                    @click="copy_tag(tag, 'top')">
                    {{ tag.en }}&nbsp
                    <n-gradient-text type="info">
                      {{ tag.zh }}&nbsp
                    </n-gradient-text>
                  </n-button>


                </n-button-group>
              </div>
            </TransitionGroup>
          </div>
        </div>
        <div class="remote_control_r">
          <n-space>
            <n-button type="info" secondary @click="copy_all">
              复制
            </n-button>
            <n-button type="error" secondary @click="close_all">
              清空
            </n-button>
          </n-space>
        </div>
      </n-card>

      <!-- 选项框 -->
      <n-card class="card_class" hoverable>
        <n-tabs type="line" animated>
          <n-tab-pane v-for="(SB, index) in StatuteBook" :name="SB.category" :tab="SB.category">
            <div v-for="t in SB.types" v-show="SB.category != 'R18' || showR18">
              <h3>{{ t.name }}</h3>
              <n-button class="tag_button" v-for="(tag, index) in t.tags" @click="to_prompt(tag)"
                :type="tag.R18 == 'false' ? 'success' : 'error'" :dashed="selected(tag)" :ghost="selected(tag)"
                :secondary="!selected(tag)" v-show="tag.R18 == 'false' || showR18">
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
  LogoWebComponent
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
  copy_tag(tag, 'low')
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
  let tags_en: any = [];
  tags.value.forEach(function (x: any) {
    tags_en.push(x.en);
  });
  prompt.value = tags_en.join(",");
}

//删除对应标签
function closeTag(tag: any) {
  let index = tags.value.indexOf(tag);
  //去掉左右两边的括号！
  if (tag.en[0] == "(" || tag.en[0] == "[" || tag.en[0] == "{") {
    tag.en = tag.en.slice(1, -1);
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
//单击复制标签！
function copy_tag(tag: any, who: any) {
  if (OpenCopy.value) {
    let exist = tags.value.includes(tag);
    if (!exist && who == 'low') {
      copy(tag.en);
      msgg.value.show();
    } else if (exist && who == 'top') {
      copy(tag.en);
      msgg.value.show();
    }
  }
}

function goUrl(url: string) {
  // location.href=url
  window.open(url)
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
const showWeight = useStorage("showWeight", true);
const showClose = useStorage("showClose", true);
const OpenCopy = useStorage("OpenCopy", false);
//是否显示18禁
// let showR18 = ref(false);
const showR18 = useStorage("showR18", false);
//切换括号
let togglebracket = ref(false);

//加重权重
function addKH(tag: any) {
  if (tag.en[0] == "(") {
    let index = tag.en.indexOf(':')
    if (index) {
      let num1 = String(tag.en[index + 1])
      let num2 = String(tag.en[index + 2])
      let num3 = String(tag.en[index + 3])
      let num = (Number(num1 + num2 + num3) * 10 + 1) / 10
      tag.en = tag.en.slice(1, -5)
      if (num3 == '9' && num < 10) {
        tag.en = "(" + tag.en + ":" + num + ".0)";
      } else if (num != 1 && num < 10) {
        tag.en = "(" + tag.en + ":" + num + ")";
      }
    }
  } else {
    tag.en = "(" + tag.en + ":1.1)";
  }
  JoinWord();
}
//减少权重
function subKH(tag: any) {
  if (tag.en[0] == "(") {
    let index = tag.en.indexOf(':')
    if (index) {
      let num1 = String(tag.en[index + 1])
      let num2 = String(tag.en[index + 2])
      let num3 = String(tag.en[index + 3])
      let num = (Number(num1 + num2 + num3) * 10 - 1) / 10
      tag.en = tag.en.slice(1, -5)
      if (num3 == '1' && num > 1) {
        tag.en = "(" + tag.en + ":" + num + ".0)";
      } else if (num != 1 && num > 0) {
        tag.en = "(" + tag.en + ":" + num + ")";
      }
    }
  } else {
    tag.en = "(" + tag.en + ":0.9)";
  }

  JoinWord();
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
  color: var(--n-text-color) !important;
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

.XGicon_tr {
  cursor: pointer;
  color: var(--n-text-color);

  &:hover {
    color: #5ae6ae;
  }
}
</style>
