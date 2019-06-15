<template>
  <div class="color-opacity">
    <div class="color-opacity-bar" ref="opacityBar" :style="background" @mousedown="barClick"></div>
    <div class="color-opacity-thumb" :style="thumbPosition" @mousedown="onMousedown"></div>
  </div>
</template>

<script>
export default {
  props: {
    showColor: {
      type: Object,
      default: () => {}
    }
  },

  data() {
    return {
      thumbPosition: {
        left: "0px"
      },
      barInfo: {}
    };
  },

  computed: {
    background() {
      let { r, g, b } = this.showColor;
      let start = `rgba(${r},${g},${b},0)`;
      let end = `rgb(${r},${g},${b})`;

      return {
        background: `linear-gradient(to right, ${start} 0%, ${end} 100%)`
      };
    }
  },

  watch: {
    showColor: {
      handler() {
        this.calcOpacity();
      },
      deep: true
    }
  },

  mounted() {
    this.calcOpacity();
  },

  methods: {
    calcOpacity() {
      let { left: l, width: w } = this.$refs.opacityBar.getBoundingClientRect();
      this.barInfo = { l, w };

      let { left } = this.thumbPosition;
      left = parseInt(left.substr(0, left.length - 2));
      this.thumbPosition.left = w * this.showColor.a + "px";
    },

    barClick($event) {
      let { l, w } = this.barInfo;
      this.thumbPosition.left = $event.offsetX + "px";

      let value = ($event.offsetX / w).toFixed(2);
      if (value == 1) value = 1;
      this.showColor.a = value;

      // 点击背景后，滑块移动到指针处，在未松开鼠标时依然可拖动
      this.onMousedown();
    },

    onMousedown() {
      let { l, w } = this.barInfo;

      document.onmousemove = r => {
        let thumbLeft = r.pageX - l;

        if (thumbLeft >= w) thumbLeft = w;
        if (thumbLeft <= 0) thumbLeft = 0;

        this.thumbPosition.l = thumbLeft + "px";

        // 透明度保留两位小数，当为1.00时为1
        let value = (thumbLeft / w).toFixed(2);
        if (value == 1) value = 1;
        this.showColor.a = value;
      };

      document.onmouseup = r => {
        document.onmousemove = null;
        document.onmouseup = null;
      };
    }
  }
};
</script>

<style lang="stylus">
.color-opacity {
  background-image: url('../img/opacityBg.png');
  background-size: 14px 14px;
  position: relative;

  .color-opacity-bar {
    width: 100%;
    height: 14px;
  }

  .color-opacity-thumb {
    position: absolute;
    top: -1px;
    left: 0;
    width: 4px;
    height: 100%;
    background: #fff;
    box-sizing: content-box;
    border: 1px solid #ddd;
    border-radius: 4px;
    cursor: pointer;
    box-shadow: 0px 0px 2px rgba(0, 0, 0, 0.6);
  }
}
</style>
