<template>
  <div class="color-select">
    <div
      class="color-select-item"
      :class="{ bgImage:it.a<1 }"
      v-for="(it, index) in list"
      :key="index"
      @click="selectColor(it)"
    >
      <div class="inner-item" :style="{background:toBg(it)}"></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    showColor: {
      type: Object,
      default: () => {}
    },
    calcBg: {
      type: Function
    }
  },

  data() {
    return {
      list: [
        { r: 255, g: 69, b: 0, a: 1 },
        { r: 255, g: 140, b: 0, a: 1 },
        { r: 255, g: 214, b: 0, a: 1 },
        { r: 144, g: 238, b: 144, a: 1 },
        { r: 0, g: 206, b: 209, a: 1 },
        { r: 30, g: 144, b: 255, a: 1 },
        { r: 199, g: 21, b: 133, a: 1 },
        { r: 255, g: 69, b: 0, a: 0.58 },
        { r: 255, g: 120, b: 0, a: 1 },
        { r: 250, g: 212, b: 0, a: 1 },
        { r: 144, g: 240, b: 144, a: 0.5 },
        { r: 0, g: 186, b: 189, a: 1 },
        { r: 31, g: 147, b: 255, a: 0.73 },
        { r: 255, g: 0, b: 0, a: 1 }
      ]
    };
  },

  methods: {
    toBg(it) {
      let { r, g, b, a } = it;
      return `rgba(${r},${g},${b},${a})`;
    },

    selectColor(it) {
      for (let key in it) {
        this.showColor[key] = it[key];
      }
      this.calcBg();
    }
  }
};
</script>

<style lang="stylus">
.color-select {
  .color-select-item {
    display: inline-block;
    margin-right: 10px;
    margin-bottom: 10px;
    cursor: pointer;

    &.bgImage {
      background-image: url('../img/opacityBg.png');
    }

    .inner-item {
      width: 24px;
      height: 24px;
      border-radius: 4px;
    }
  }
}
</style>
