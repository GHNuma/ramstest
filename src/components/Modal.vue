<template>
  <div class="modal" v-if="isVisible">
    <div class="modal-content">
      <img :src="imageUrl" alt="Modal Image" />
    </div>
    <div class="close-container" @click="hideModal" @touchend="hideModal">
      <img src="~@/assets/icons/close.svg" alt="close" />
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isVisible: Boolean,
    imageUrl: String,
  },
  methods: {
    hideModal(e) {
      e.stopPropagation();
      this.$emit('close');
    },
  },
};
</script>

<style>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 1000;
  cursor: pointer;
}
.modal {
  cursor: default;
}
.modal-content {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal img {
  max-width: 100%;
  max-height: 100%;
  pointer-events: none;
}

.close-container {
  position: absolute;
  width: 60px;
  height: 60px;
  top: 20px;
  right: 20px;
  cursor: pointer;
  transition: all 0.3s;
}
.close-container:hover {
  transform: scale(0.9);
}
/*По фану*/
.close-container:active {
  transition: all 0.1ms;
  transform: scale(1.05);
}
</style>
