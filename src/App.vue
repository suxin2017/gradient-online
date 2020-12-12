<template>
  <div class="preview" :style="{ background: preview.gradint }"></div>
  <div class="select-panel">
    <div class="code-preview">
      <pre class="code"
        >{{ preview.gradint }}
    </pre
      >
      <div class="copy" @click="copyCode(preview.gradint)">copy</div>
    </div>
    <div
      class="select-panel-banner"
      :style="{ background: state.selectPanelGradint }"
    >
      <ColorButton
        v-for="button in state.colorButtons"
        :key="button"
        :color="button.color"
        :initLeft="button.initLeft"
        :active="button.active"
        :zIndex="button.zIndex"
        @onMove="buttonMove(button, $event)"
        @click="setActiveButton(button)"
      />
    </div>

    <div class="select-panel-color">
      <div class="form-group">渐变配置</div>
      <div class="form-item">
        <label> 渐变类型 </label>
        <select v-model="preview.gradientType">
          <option value="linear-gradient">线性渐变-</option>
          <option value="radial-gradient">镜像渐变</option>
        </select>
      </div>
      <div class="form-item" v-if="preview.gradientType === 'linear-gradient'">
        <label> 渐变角度 </label>
        <input type="number" v-model="preview.lineDeg" />
      </div>
      <div class="form-item" v-if="preview.gradientType === 'radial-gradient'">
        <label> 形状 </label>
        <select v-model="preview.radialShape">
          <option value="ellipse">ellipse 椭圆形-</option>
          <option value="circle">circle 圆形</option>
        </select>
      </div>
      <div class="form-item" v-if="preview.gradientType === 'radial-gradient'">
        <label> 大小 </label>
        <select v-model="preview.radialSize">
          <option value="closest-side">closest-side</option>
          <option value="farthest-side">farthest-side</option>
          <option value="closest-corner">closest-corner</option>
          <option value="farthest-corner">farthest-corner</option>
        </select>
      </div>
      <div class="form-item" v-if="preview.gradientType === 'radial-gradient'">
        <label> 中心X </label>
        <input type="number" v-model="preview.radialCenterX" />
      </div>
      <div class="form-item" v-if="preview.gradientType === 'radial-gradient'">
        <label> 中心Y </label>
        <input type="number" v-model="preview.radialCenterY" />
      </div>

      <div class="form-group">渐变按钮</div>
      <div class="form-item">
        <label> 按钮操作 : </label>
        <input
          class="form-item-button"
          type="button"
          value="添加按钮"
          @click="addButton"
        />
        <input
          class="form-item-button"
          type="button"
          value="删除按钮"
          @click="deleteButton"
        />
      </div>
      <div class="form-item">
        <label> 颜色 : </label>
        <input type="color" v-model="state.activeButton.color" />
      </div>
    </div>

    <div class="debugger" v-if="debug">
      <pre>
当前选择颜色 {{ state.activeButton }} preview样式
{{ preview.gradint }}
{{ state.selectPanelGradint }}
      </pre>
      <div v-for="button in state.colorButtons" :key="button">
        color: {{ button.color }} offset: {{ button.offset }}
      </div>
      <div>
        {{ preview.lineDeg }}
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, reactive, watch } from "vue";
import ColorButton from "./components/ColorButton.vue";
import { fallbackCopyTextToClipboard } from "./util";
export const SELECT_WIDTH = 550;
const defaultColor = [
  { color: "#ff0000", offset: 0, initLeft: 0, key: 0 },
  { color: "#ffffff", offset: 0.5, initLeft: SELECT_WIDTH * 0.5, key: 1 },
  { color: "#00ff00", offset: 1, initLeft: SELECT_WIDTH, key: 2 },
];
let UID = 3;
export default {
  name: "App",
  components: {
    ColorButton,
  },
  setup() {
    const state = reactive({
      colorButtons: defaultColor,
      activeButton: defaultColor[0],
      selectPanelGradint: "",
    });
    const preview = reactive({
      gradint: "",
      lineDeg: 180,
      radialCenterX: 50,
      radialCenterY: 50,
      radialShape: "ellipse",
      radialSize: "farthest-corner",
      gradientType: "linear-gradient",
    });

    const selectBarWidth = SELECT_WIDTH;

    function setSelectPanelGradint() {
      state.selectPanelGradint = generaryLinearGradient(90, "linear-gradient");
    }
    function setPreviewGradint() {
      preview.gradint = generaryLinearGradient(
        preview.lineDeg,
        preview.gradientType
      );
    }
    watch([preview, state.colorButtons], () => {
      setSelectPanelGradint();
      setPreviewGradint();
    });

    onMounted(() => {
      setSelectPanelGradint();
      setPreviewGradint();
    });

    function buttonMove(button, x) {
      const offset = x / selectBarWidth;
      button.offset = offset.toFixed(2);
    }

    function generaryLinearGradient(deg, type) {
      if (type === "linear-gradient") {
        let result = `linear-gradient(${deg}deg,`;
        result += state.colorButtons
          .sort((x, y) => x.offset - y.offset)
          .reduce((a, b) => {
            if (a !== "") {
              return a + `, ${b.color} ${(b.offset * 100).toFixed(2)}%`;
            }
            return a + `${b.color} ${(b.offset * 100).toFixed(2)}%`;
          }, "");
        result += ")";
        return result;
      } else {
        let result = `radial-gradient(${preview.radialShape} ${preview.radialSize} at ${preview.radialCenterX}% ${preview.radialCenterY}% ,`;
        result += state.colorButtons
          .sort((x, y) => x.offset - y.offset)
          .reduce((a, b) => {
            if (a !== "") {
              return a + `, ${b.color} ${(b.offset * 100).toFixed(2)}%`;
            }
            return a + `${b.color} ${(b.offset * 100).toFixed(2)}%`;
          }, "");
        result += ")";
        return result;
      }
    }
    function addButton() {
      let activeIndex = state.colorButtons.findIndex(
        (button) => button.key === state.activeButton.key
      );
      const newButton = { ...state.activeButton };
      newButton.key = UID++;
      state.colorButtons.splice(activeIndex, 0, newButton);
      setActiveButton(newButton);
    }
    function deleteButton() {
      let activeIndex = state.colorButtons.findIndex(
        (button) => button.key === state.activeButton.key
      );
      state.colorButtons.splice(activeIndex, 1);
      setActiveButton(state.colorButtons[0]);
    }
    function setActiveButton(button) {
      state.colorButtons.forEach((item) => {
        item.active = false;
        button.zIndex = 0;
      });
      button.active = true;
      button.zIndex = 1;
      state.activeButton = button;
    }
    return {
      addButton,
      setActiveButton,
      buttonMove,
      deleteButton,
      preview,
      state,
      copyCode: fallbackCopyTextToClipboard,
      debug: process.env.NODE_ENV === "development",
    };
  },
};
</script>

<style>
* {
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.preview {
  width: 600px;
  height: 400px;
  border: 1px #c1c1c1 solid;
  border-radius: 12px;
  margin: 0 auto;
}
.code-preview {
  position: relative;
}
.copy {
  position: absolute;
  right: 5px;
  top: 5px;
  font-size: 12px;
  border: 1px solid;
  border-radius: 5px;
  padding: 2px 5px;
  color: #666;
  cursor: pointer;
}
.code {
  white-space: pre-wrap;
  background: #efefef85;
  padding: 25px 5px 0px;
  line-height: 25px;
  border-radius: 10px;
}
.select-panel {
  width: 600px;
  border: 1px #c1c1c1 solid;
  border-radius: 12px;
  margin: 20px auto;
  padding: 20px;
}
.select-panel-banner {
  border: 1px #c1c1c1 solid;
  border-radius: 4px;
  height: 20px;
  position: relative;
}
.select-panel-color {
  text-align: left;
}
.form-group {
  font-weight: 600;
  margin: 20px 0;
}
.form-item {
  margin: 20px 0;
}
.form-item-button + .form-item-button {
  margin-left: 20px;
}
</style>
