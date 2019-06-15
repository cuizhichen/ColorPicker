<template>
  <div class="color-sidebar">
    <div class="side-bar-bg" ref="sideBarBg" @mousedown="barClick"></div>
    <div class="side-bar-thumb" :style="thumbPosition" @mousedown="onMousedown"></div>
  </div>
</template>

<script>
export default {
  props: {
    bgColor: {
      type: Object,
      default: () => {}
    }
  },

  data() {
    return {
      thumbPosition: {
        top: "0px"
      },
      sideBarHeight: null
    };
  },

  mounted() {
    this.sideBarHeight = this.$refs.sideBarBg.getBoundingClientRect().height;
  },

  methods: {
    onMousedown() {
      let { top: t, height: h } = this.$refs.sideBarBg.getBoundingClientRect();

      document.onmousemove = r => {
        let top = r.pageY - t;
        if (top >= h) top = h;
        if (top <= 0) top = 0;

        this.changeBg(top, h);
      };

      document.onmouseup = r => {
        document.onmousemove = null;
        document.onmouseup = null;
      };
    },

    // 修改背景图
    changeBg(top, h) {
      // 修改前先将滑块的位置改变
      this.thumbPosition.top = top + "px";

      // 侧栏一共分为六个区域，每块区域的长度
      let total = h / 6;

      if (top <= (h * 1) / 6) {
        let g = this.getValue(top, total, 0);
        this.setColor(255, g, 0);
      } else if (top <= (h * 2) / 6) {
        let r = this.getValue(top, total, 1);
        this.setColor(255 - r, 255, 0);
      } else if (top <= (h * 3) / 6) {
        let b = this.getValue(top, total, 2);
        this.setColor(0, 255, b);
      } else if (top <= (h * 4) / 6) {
        let g = this.getValue(top, total, 3);
        this.setColor(0, 255 - g, 255);
      } else if (top < (h * 5) / 6) {
        let r = this.getValue(top, total, 4);
        this.setColor(r, 0, 255);
      } else if (top <= (h * 6) / 6) {
        let b = this.getValue(top, total, 5);
        this.setColor(255, 0, 255 - b);
      }

      // 通知父级组件调用ColorPanel修改当前选中的颜色
      this.$emit("changeShow");
    },

    barClick($event) {
      let height = this.sideBarHeight;
      let top = $event.offsetY;

      this.changeBg(top, height);

      // 点击背景后，滑块移动到指针处，在未松开鼠标时依然可拖动
      this.onMousedown();
    },

    getValue(top, total, index) {
      let value = ((top - total * index) / total) * 255;
      let int = parseInt(value);
      return int;
    },

    setColor(r, g, b) {
      this.bgColor.r = r;
      this.bgColor.g = g;
      this.bgColor.b = b;
    },

    // 通过选择或输入背景颜色修改时，重新计算calcTop
    calcTop() {
      let { r, g, b } = this.bgColor;
      let height = this.sideBarHeight;

      let top = 0;
      let total = height / 6;

      if (r === 255 && b === 0) top = (g / 255) * total;
      if (g === 255 && b === 0) top = (r / 255 + 1) * total;
      if (g === 255 && r === 0) top = (b / 255 + 2) * total;
      if (b === 255 && r === 0) top = (g / 255 + 3) * total;
      if (b === 255 && g === 0) top = (r / 255 + 4) * total;
      if (r === 255 && g === 0) top = (b / 255 + 5) * total;

      this.thumbPosition.top = top + "px";
    }
  }
};
</script>

<style lang="stylus">
.color-sidebar {
  position: relative;

  .side-bar-bg {
    width: 100%;
    height: 100%;
    background: linear-gradient(180deg, red 0, #ff0 17%, #0f0 33%, #0ff 50%, #00f 67%, #f0f 83%, red);
  }

  .side-bar-thumb {
    position: absolute;
    top: 0;
    left: -1px;
    width: 100%;
    height: 4px;
    background: #fff;
    box-sizing: content-box;
    border: 1px solid #ddd;
    border-radius: 4px;
    cursor: pointer;
    box-shadow: 0px 0px 2px rgba(0, 0, 0, 0.6);
  }
}
</style>
