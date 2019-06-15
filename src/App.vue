<template>
  <div id="app">
    <button @click="show = !show">{{ show ? '隐藏' : '显示' }}</button>
    <button @click="switchFormat">切换颜色格式</button>
    <div class="current-color">当前颜色：{{color}}</div>
    <ck-color-picker v-model="color" :color-format="format" :show.sync="show"/>
  </div>
</template>

<script>
import CkColorPicker from "@/components/ColorPicker";

export default {
  name: "app",

  components: {
    CkColorPicker
  },

  data() {
    return {
      color: "#c83aaa",
      show: true,
      format: "hex"
    };
  },

  methods: {
    switchFormat() {
      this.format = this.format === "hex" ? "rgb" : "hex";
    }
  }
};
</script>

<style lang='stylus'>
$mc = #409eff;

#app {
  margin: 50px;

  button {
    background: #fff;
    padding: 4px 16px;
    margin-bottom: 12px;
    border: 1px solid #bbbbbb;
    border-radius: 4px;
    font-size: 15px;
    cursor: pointer;
    outline: none;
    margin-right: 20px;
    color: #555;

    &:hover {
      border-color: $mc;
      color: $mc;
    }
  }

  .current-color {
    margin-bottom: 24px;
  }
}
</style>
