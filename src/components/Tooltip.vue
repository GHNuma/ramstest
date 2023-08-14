<template>
  <div class="tooltip" :class="{ show: isTooltipVisible }" :style="tooltipStyle">
    {{ content }}
  </div>
</template>

<script>
export default {
  props: {
    content: String,
    top: String,
    left: String,
    right: String,
    bottom: String,
    isTooltipVisible: Boolean,
  },
  computed: {
    tooltipStyle() {
      return {
        top: this.top,
        left: this.left,
        right: this.right,
        bottom: this.bottom,
      };
    },
  },
};
</script>

<style>
.tooltip {
  position: absolute;
  padding: 5px;
  background-color: sandybrown;
  color: white;
  border-radius: 8px;
  font-size: 16px;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s ease;
  z-index: 2;
  transform: translate(-50%, 0);
}

.tooltip.show {
  opacity: 1;
}
</style>
