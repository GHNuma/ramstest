<template>
  <div
    v-if="isVisible"
    class="hotspot"
    :style="hotspotStyle"
    @mouseenter="(e) => showTooltip(e, index)"
    @mouseleave="hideTooltip"
    @click="handleClick"
    @touchstart="(e) => showTooltip(e, index)"
    @touchend="handleTouchEnd"
    @touchcancel="handleTouchEnd"
  >
    <img class="hotspot-icon" src="~@/assets/icons/appartment.svg" alt="Hotspot" />
  </div>
</template>

<script>
const hotspotProps = {
  isVisible: Boolean,
  top: String,
  left: String,
  index: String,
  showTooltip: Function,
  hideTooltip: Function,
  touchIsActive: false,
  touchholdTimer: null,
};

export default {
  props: hotspotProps,
  computed: {
    hotspotStyle() {
      return {
        top: this.top,
        left: this.left,
      };
    },
  },
  methods: {
    openModal() {
      this.$emit('open-modal', this.index);
    },
    handleClick() {
      if (!this.isModalOpen) {
        this.openModal();
      }
    },
    handleTouchEnd() {
      this.hideTooltip();
      if (!this.isModalOpen) {
        this.openModal();
      }
      clearTimeout(this.touchholdTimer);
    },
  },
};
</script>

<style>
.hotspot {
  position: absolute;
  z-index: 1;
  touch-action: none;
}

.hotspot-icon {
  width: 30px;
  height: 30px;
  user-select: none;
  cursor: pointer;
  transition: all 0.5s;
}
</style>
