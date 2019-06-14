<template>
  <div class="color-panel" ref="panelBg" :style="{background}" @mousedown.stop="bgClick">
    <div class="panel-bg-white"></div>
    <div class="panel-bg-black"></div>
    <div class="panel-thumb" :style="thumbPosition" @mousedown.stop="onMousedown"></div>
  </div>
</template>

<script>
export default {
  props: {
    bgColor: {
      type: Object,
      default: () => {}
    },

    showColor: {
      type: Object,
      default: () => {}
    }
  },

  data() {
    return {
      thumbPosition: {
        left: "0px",
        top: "0px"
      },
      panelBgInfo: {}
    };
  },

  computed: {
    background() {
      let { r, g, b, a } = this.bgColor;
      return `rgba(${r}, ${g}, ${b}, ${a})`;
    }
  },

  mounted() {
    this.calcXY();
  },

  methods: {
    // 基于当前显示的颜色移动滑块
    calcXY() {
      let {
        left: l,
        top: t,
        width: w,
        height: h
      } = this.$refs.panelBg.getBoundingClientRect();
      this.panelBgInfo = { l, t, w, h };

      let { r, g, b } = this.showColor;
      let max = Math.max(r, g, b);
      let min = Math.min(r, g, b);
      let x = (min / max) * w;
      let y = h - (max / 255) * h;

      this.thumbPosition.left = w - x + "px";
      this.thumbPosition.top = y + "px";
    },

    // 当滑块被点击时
    onMousedown() {
      let { l, t, w, h } = this.panelBgInfo;

      document.onmousemove = res => {
        let { pageX, pageY } = res;
        let left = pageX - l;
        let top = pageY - t;

        if (left <= 0) left = 0;
        if (left >= w) left = w;
        if (top <= 0) top = 0;
        if (top >= h) top = h;

        this.thumbPosition = {
          left: left + "px",
          top: top + "px"
        };

        let x = w - left;
        let y = top;

        this.calcWidthColor(x, y);
      };

      document.onmouseup = r => {
        document.onmousemove = null;
        document.onmouseup = null;
      };
    },

    // 此函数为父级组件调用
    // 背景颜色修改时，需基于当前滑块重新计算当前选中的颜色
    changeShowColor() {
      let { l, t, w, h } = this.panelBgInfo;
      let { left, top } = this.thumbPosition;

      left = parseInt(left.substr(0, left.length - 2));
      top = parseInt(top.substr(0, top.length - 2));

      let x = w - left;
      let y = top;

      this.calcWidthColor(x, y, w, h);
    },

    calcWidthColor(x, y) {
      let { w, h } = this.panelBgInfo;

      let newColor = JSON.parse(JSON.stringify(this.bgColor));
      delete newColor.a;

      for (let key in newColor) {
        // hue => 色相、三原色
        let hue = newColor[key];
        let value = ((255 - hue) * x + hue * w) / w;
        newColor[key] = value;
      }

      this.calcHeightColor(x, y, w, h, newColor);
    },

    // 通过右边界交点计算原点的坐标
    calcHeightColor(x, y, w, h, color) {
      for (let key in color) {
        // hue => 色相、三原色
        let hue = color[key];
        let value = ((h - y) * hue) / h;
        let int = parseInt(value);

        this.showColor[key] = int;
      }
    },

    // 点击背景时，滑块移动到点击的地方
    bgClick($event) {
      // 防止滑块触发背景点击事件
      if ($event.target.className === "panel-thumb") return;

      let { offsetX, offsetY } = $event;
      let { w } = this.panelBgInfo;

      // 基于点击的位置给滑块定位
      this.thumbPosition = {
        left: offsetX + "px",
        top: offsetY + "px"
      };

      // 因为是基于右上角给推断颜色，所以将颜色的原点设在右上角
      // 所以此处x需要用宽度减去滑块的left
      let x = w - offsetX;
      let y = offsetY;

      this.calcWidthColor(x, y);

      // 点击背景后，滑块移动到指针处，在未松开鼠标时依然可拖动
      this.onMousedown();
    }
  }
};
</script>

<style lang="stylus">
.color-panel {
  position: relative;

  .panel-bg-white, .panel-bg-black {
    width: 100%;
    height: 100%;
    position: absolute;
  }

  .panel-bg-white {
    background: linear-gradient(90deg, #fff, hsla(0, 0%, 100%, 0));
  }

  .panel-bg-black {
    background: linear-gradient(0deg, #000, transparent);
  }

  .panel-thumb {
    position: absolute;
    width: 5px;
    height: 5px;
    box-shadow: 0 0 0 1.5px #fff, inset 0 0 1px 1px rgba(0, 0, 0, 0.3), 0 0 1px 2px rgba(0, 0, 0, 0.4);
    border-radius: 50%;
    transform: translate(-2px, -2px);
    cursor: pointer;
  }
}
</style>