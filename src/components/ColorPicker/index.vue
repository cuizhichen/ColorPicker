<template>
  <transition name="colorPicker">
    <div class="color-picker" ref="colorPicker" v-show="show">
      <div class="color-picker-inner">
        <div class="color-picker-up">
          <color-panel ref="colorPanel" :bgColor="bgColor" :showColor="showColor"/>
          <color-sidebar ref="colorSidebar" :bgColor.sync="bgColor" @changeShow="changeShow"/>
        </div>
        <color-opacity :showColor="showColor"/>
        <color-select :showColor="showColor" :calcBg="calcBg"/>
        <div class="color-picker-down">
          <div class="color-number">
            <input type="text" :value="colorToRgba" @change="onChange">
          </div>
          <div class="color-random" @click="colorRandom">随机</div>
          <div class="color-confirm" @click="$emit('update:show', false)">确定</div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import {
  ColorPanel,
  ColorSidebar,
  ColorOpacity,
  ColorSelect
} from "./components/index";

export default {
  components: {
    ColorPanel,
    ColorSidebar,
    ColorOpacity,
    ColorSelect
  },

  model: {
    prop: "value",
    event: "change"
  },

  props: {
    value: {
      type: String,
      default: "#c83aaa"
    },
    show: {
      type: Boolean,
      default: true
    },
    colorFormat: {
      type: String,
      default: "hex"
    }
  },

  data() {
    return {
      bgColor: {
        r: null,
        g: null,
        b: null,
        a: 1
      },

      showColor: {
        r: null,
        g: null,
        b: null,
        a: null
      },

      once: true
    };
  },

  computed: {
    colorToRgba() {
      if (this.showColor.r === null) return;
      // 根据颜色格式返回数值
      if (this.colorFormat === "hex") {
        var value = this.RgbToHex();
      } else {
        let { r, g, b, a } = this.showColor;
        var value = `rgba(${r},${g},${b},${a})`;
      }
      value = value.replace(/\s+/g, "");

      // 动态修改颜色格式时触发一次修改事件
      if (this.value !== value) this.$emit("change", value);

      return value;
    }
  },

  watch: {
    showColor: {
      handler() {
        // 防止第一次传参时就触发change事件
        if (this.once) {
          this.once = false;
          return;
        }

        // 根据颜色格式返回数值
        if (this.colorFormat === "hex") {
          var value = this.RgbToHex();
        } else {
          let { r, g, b, a } = this.showColor;
          var value = `rgba(${r},${g},${b},${a})`;
        }

        this.$emit("change", value);
      },
      deep: true
    }
  },

  mounted() {
    let height = this.$refs.colorPicker.scrollHeight;
    this.$refs.colorPicker.style.setProperty("--size", height + "px");
    // 第一次加载时将数值格式化
    this.onChange();
  },

  methods: {
    // 当输入框的值改变时
    onChange($el) {
      // 判断在何时调用的此方法
      // 将输入的所有空格去除
      let value = $el ? $el.target.value : this.value;

      value = value.replace(/\s+/g, "");

      // 如果输入的第一位是#，则输入的是16进制的颜色
      if (value[0] === "#") {
        var list = this.HexToRgb(value);
      } else {
        value = value.replace(/[a-zA-Z\(\)]/g, "");
        var list = value.split(",");
      }

      // 如果数组不存在；或者长度不为3或4则输入不正确
      if (!list || (list.length !== 3 && list.length !== 4)) {
        console.error("数值输入错误");
        return;
      }

      this.showColor.r = this.calcColorNumber(list[0]);
      this.showColor.g = this.calcColorNumber(list[1]);
      this.showColor.b = this.calcColorNumber(list[2]);
      this.showColor.a = list[3] || 1;

      // 修改完当前颜色，重新计算背景色
      this.calcBg();
    },

    // 判断是否为16进制，返回一个色号rgb数组，exm:[255,0,0]
    HexToRgb(value) {
      let color = value.substring(1, value.length);

      // 判断长度是否符合16进制色号的要求
      if (color.length !== 3 && color.length !== 6) return;

      // 判断是否为16进制
      if (/[^0-9a-f]/g.test(color)) return;

      // 判读是#000 & #000000 格式
      // 将其转换成rgb色号数组
      if (color.length === 3) {
        // 根据不同的格式进行切割和数组映射
        let value = color.replace(/\w(?=(\w{1})+$)/g, "$&,");

        var list = value.split(/,/g);
        list = list.map(e => {
          return parseInt(e + e, 16);
        });
      } else {
        let value = color.replace(/\w(?=(\w{2})+$)/g, "$&,");
        var list = value.split(/,/g);
        list = list.map(e => {
          return parseInt(e, 16);
        });
      }

      return list;
    },

    RgbToHex() {
      let value = "#";
      for (let key in this.showColor) {
        if (key === "a") continue;
        if (this.showColor[key] === null) return;

        let v = parseInt(this.showColor[key]).toString(16);
        if (v.length === 1) v += v;
        value += v;
      }

      return value;
    },

    // 判断输入的数值是否超出了范围
    calcColorNumber(cn) {
      if (cn > 255) cn = 255;
      if (cn < 0) cn = 0;
      return cn;
    },

    // 改变颜色时，计算出背景颜色
    // 背景色修改完成后调用子组件修改color-panel和color-sidebar的位置
    calcBg() {
      let { r, g, b } = this.showColor;
      let newColor = [
        { name: "r", value: r },
        { name: "g", value: g },
        { name: "b", value: b }
      ];

      newColor.sort((a, b) => {
        return b.value - a.value;
      });

      let max = newColor[0].value;
      let mid = newColor[1].value;
      let min = newColor[2].value;

      // 计算点像右延长至边界的点的色标
      newColor[1].value = ((min - mid) * max) / (min - max);
      newColor[2].value = 0;

      newColor[0].value = 255;
      newColor[1].value = parseInt((255 * newColor[1].value) / max);

      newColor.forEach(e => {
        this.bgColor[e.name] = e.value;
      });

      this.$refs.colorPanel.calcXY();

      // 计算完成需对sidebar组件滑块的位置重新定位
      this.$refs.colorSidebar.calcTop();
    },

    // 当sidebar组件改变背景色时，调用colorPanel组件的方法，计算当前指针所在的颜色
    changeShow() {
      this.$refs.colorPanel.changeShowColor();
    },

    // 颜色随机功能 透明度默认为1
    colorRandom() {
      for (let key in this.showColor) {
        this.showColor[key] = parseInt(Math.random() * 255);
        if (key === "a") this.showColor[key] = 1;
      }

      // 修改完当前颜色，重新计算背景色
      this.calcBg();
    }
  }
};
</script>

<style lang="stylus">
$mc = #409eff;

.color-picker {
  font-size: 13px;
  width: 380px;
  border: 1px solid #e9e9e9;
  border-radius: 6px;
  box-shadow: 0px 0px 12px rgba(0, 0, 0, 0.1);

  .color-picker-inner {
    padding: 12px;

    .color-picker-up {
      display: flex;
      height: 204px;
      margin-bottom: 14px;

      .color-panel {
        width: 92%;
        margin-right: 14px;
      }

      .color-sidebar {
        flex: 1;
      }
    }

    .color-opacity, .color-select {
      width: 92%;
    }

    .color-opacity {
      margin-bottom: 16px;
    }

    .color-select {
      margin-bottom: 20px;
    }

    .color-picker-down {
      display: flex;
      justify-content: space-between;
      align-items: center;

      .color-number {
        width: 50%;

        input {
          width: 100%;
          padding: 6px 8px;
          border: 1px solid #dddddd;
          outline: none;
          border-radius: 4px;
          color: #777;
          letter-spacing: 1px;
        }
      }

      .color-random {
        margin-left: auto;
        margin-right: 16px;
        color: $mc;
        cursor: pointer;

        &:hover {
          text-decoration: underline;
        }
      }

      .color-confirm {
        padding: 8px 16px;
        border: 1px solid #ddd;
        border-radius: 4px;
        cursor: pointer;

        &:hover {
          color: $mc;
          border-color: $mc;
        }
      }
    }
  }
}

.colorPicker-enter-active, .colorPicker-leave-active {
  transition: all 0.3s linear;
  overflow: hidden;
}

.colorPicker-enter, .colorPicker-leave-to {
  height: 0;
}

.colorPicker-enter-to, .colorPicker-leave {
  height: var(--size);
}
</style>
