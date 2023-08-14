<template>
  <div class="panorama-viewer">
    <div
      ref="panoramaContainer"
      class="panorama-container"
      @mousedown="startDrag"
      @mousemove="handleDrag"
      @mouseup="endDrag"
      @touchstart="startDrag"
      @touchmove="handleDrag"
      @touchend="endDrag"
      :style="{ cursor: isDragging ? 'grabbing' : 'grab' }"
    >
      <div class="panorama-image-container">
        <img
          class="panorama-image no-pointer-events"
          ref="panoramaImage"
          :src="imageUrl"
          alt="Panorama View"
        />
        <div class="hotspot-container">
          <Hotspot
            v-for="(hotspot, hotspotIndex) in hotspotsSettings"
            :key="hotspotIndex"
            :index="hotspotIndex"
            :showTooltip="showTooltip"
            :hideTooltip="hideTooltip"
            :isVisible="currentImageIndex === imageChangeIndices[hotspotIndex]"
            :top="hotspot.top"
            :left="hotspot.left"
            @open-modal="openModal"
          />
        </div>
        <div
          class="arrow-buttons"
          @mouseenter="blockDragging = true"
          @mouseleave="blockDragging = false"
          @touchstart="blockDragging = true"
          @touchend="blockDragging = false"
        >
          <button
            class="arrow-button"
            @mouseenter="handleButtonMouseEnter"
            @click="startImageChange(-1)"
            @touchend="startImageChange(-1)"
          >
            <img src="~@/assets/icons/arrow-left.svg" alt="arrow-left" />
          </button>
          <button
            class="arrow-button"
            @mouseenter="handleButtonMouseEnter"
            @click="startImageChange(1)"
            @touchend="startImageChange(1)"
          >
            <img src="~@/assets/icons/arrow-right.svg" alt="arrow-right" />
          </button>
        </div>
        <Tooltip
          v-if="isTooltipVisible"
          :content="tooltipContent"
          :top="tooltipPosition.top"
          :left="tooltipPosition.left"
          :isTooltipVisible="isTooltipVisible"
        />
        <Modal :isVisible="isModalVisible" :imageUrl="modalImageUrl" @close="closeModal" />
      </div>
    </div>
  </div>
</template>

<script>
import Tooltip from '@/components/Tooltip.vue';
import Modal from '@/components/Modal.vue';
import Hotspot from '@/components/Hotspot.vue';
export default {
  components: {
    Hotspot,
    Tooltip,
    Modal,
  },
  data() {
    return {
      panoramaImages: Array.from(
        { length: 119 },
        (_, index) => `/src/assets/panorama_sides/${index}.jpg`
      ),
      currentImageIndex: 18,
      isDragging: false,
      startX: 0,
      dragThreshold: 0,
      intervalId: null,
      imageChangeIndices: [18, 49, 86],
      imageChangeDirection: 0,
      deltaXDistance: 0,
      focusingImageIndex: 0,
      correctionProcessing: false,
      isTooltipVisible: false,
      tooltipContent: '',
      tooltipPosition: { top: '0', left: '0' },
      tooltipSettings: {
        0: {
          text: 'BlockHall',
        },
        1: {
          text: 'Hall',
        },
        2: {
          text: 'RamsEvo',
        },
      },
      hotspotsSettings: {
        0: {
          top: '50%',
        },
        1: {
          top: '60%',
        },
        2: {
          top: '30%',
          left: '85%',
        },
      },
      imageSettings: {
        0: {
          imageName: 'BlockHall',
        },
        1: {
          imageName: 'Hall',
        },
        2: {
          imageName: 'RamsEvo',
        },
      },
      isModalVisible: false,
      modalImageUrl: '',
      isModalOpen: false,
      blockDragging: false,
    };
  },
  methods: {
    initPanoramaViewer() {
      const panoramaImage = this.$refs.panoramaImage;

      let isDragging = false;
      let previousX = 0;

      panoramaImage.addEventListener('mousedown', (e) => {
        isDragging = true;
        previousX = e.clientX;
      });

      document.addEventListener('mousemove', (e) => {
        if (!isDragging) return;

        const deltaX = e.clientX - previousX;
        previousX = e.clientX;

        const rotation = parseFloat(
          getComputedStyle(panoramaImage).getPropertyValue('transform').split(',')[4]
        );
        panoramaImage.style.transform = `translateX(-50%) rotateY(${rotation + deltaX * 0.1}deg)`;
      });

      document.addEventListener('mouseup', () => {
        isDragging = false;
      });

      document.addEventListener('mouseleave', () => {
        isDragging = false;
      });
    },
    changeImage(direction) {
      this.currentImageIndex =
        (this.currentImageIndex + direction + this.panoramaImages.length) %
        this.panoramaImages.length;
    },
    startImageChange(direction) {
      this.stopRepeatChangeImage();
      this.imageChangeDirection = direction > 0 ? 0 : 1;
      this.intervalId = setInterval(this.changeImageWithInterval, 15);
      this.correctionProcessing = true;
    },
    changeImageWithInterval() {
      if (this.imageChangeDirection === 0) {
        this.changeImage(1);
      } else {
        this.changeImage(-1);
      }
      if (this.imageChangeIndices.includes(this.currentImageIndex)) {
        this.stopRepeatChangeImage();
        this.correctionProcessing = false;
        this.focusingImageIndex = this.imageChangeIndices.indexOf(this.currentImageIndex);
      }
    },
    stopRepeatChangeImage() {
      clearInterval(this.intervalId);
    },
    startDrag(event) {
      event.preventDefault();
      if (!this.isModalOpen && !this.blockDragging) {
        event.preventDefault();
        const clientX = event.type === 'touchstart' ? event.touches[0].clientX : event.clientX;
        const clientY = event.type === 'touchstart' ? event.touches[0].clientY : event.clientY;

        this.isDragging = !this.correctionProcessing;
        this.startX = clientX;
        this.startY = clientY;
        this.deltaXDistance = 0;
      }
    },

    handleDrag(event) {
      event.preventDefault();
      if (!this.isModalOpen && !this.blockDragging && this.isDragging) {
        event.preventDefault();

        const clientX = event.type === 'touchmove' ? event.touches[0].clientX : event.clientX;
        const clientY = event.type === 'touchmove' ? event.touches[0].clientY : event.clientY;

        const deltaX = clientX - this.startX;
        this.startX = clientX;
        const deltaY = clientY - this.startY;
        this.startY = clientY;

        const rotation = parseFloat(
          getComputedStyle(this.$refs.panoramaImage).getPropertyValue('transform').split(',')[4]
        );
        this.$refs.panoramaImage.style.transform = `translateX(-50%) rotateY(${
          rotation + deltaX * 0.1
        }deg)`;

        if (Math.abs(deltaX) > this.dragThreshold || Math.abs(deltaY) > this.dragThreshold) {
          if (deltaX > 0) {
            this.changeImage(1);
            this.deltaXDistance++;
          } else {
            this.changeImage(-1);
            this.deltaXDistance--;
          }
        }
      }
    },
    endDrag() {
      if (!this.isModalOpen) {
        if (this.deltaXDistance === 0) {
          this.isDragging = false;
        }
        if (this.isDragging === false) {
          return;
        }
        this.isDragging = false;
        this.stopRepeatChangeImage();
        let distance;
        let direction;
        if (this.deltaXDistance > 0) {
          if (this.focusingImageIndex !== this.imageChangeIndices.length - 1) {
            distance =
              this.imageChangeIndices[this.focusingImageIndex + 1] -
              this.imageChangeIndices[this.focusingImageIndex];
          } else {
            distance = this.imageChangeIndices[0];
          }
          direction = this.deltaXDistance < distance / 2 ? 0 : 1;
        } else if (this.deltaXDistance < 0) {
          if (this.focusingImageIndex !== 0) {
            distance =
              this.imageChangeIndices[this.focusingImageIndex] -
              this.imageChangeIndices[this.focusingImageIndex - 1];
          } else {
            distance = this.imageChangeIndices[this.focusingImageIndex];
          }
          direction = -1 * this.deltaXDistance < distance / 2 ? 1 : 0;
        }
        this.startImageChange(direction);
      }
    },
    handleButtonMouseEnter() {
      if (this.isDragging) {
        this.endDrag();
      }
    },
    showTooltip(event, index) {
      const hotspotRect = event.target.getBoundingClientRect();
      this.isTooltipVisible = true;
      this.tooltipContent = this.tooltipSettings[index].text;
      this.tooltipPosition = {
        top: `${hotspotRect.top - hotspotRect.height}px`,
        left: `${hotspotRect.left + hotspotRect.width / 2}px`,
      };
    },
    hideTooltip() {
      this.isTooltipVisible = false;
    },
    openModal(index) {
      this.modalImageUrl = `/src/assets/appartment_images/${this.imageSettings[index].imageName}.jpg`;
      this.isModalVisible = true;
      this.isModalOpen = true;
    },

    closeModal() {
      this.isModalVisible = false;
      this.isModalOpen = false;
    },
  },
  computed: {
    imageUrl() {
      return this.panoramaImages[this.currentImageIndex];
    },
  },
  mounted() {
    this.initPanoramaViewer();
  },
};
</script>

<style>
.panorama-viewer {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  user-select: none;
}

.panorama-container {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  user-select: none;
}

.panorama-image-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  width: 100%;
  height: 100%;
  user-select: none;
}

.panorama-image.no-pointer-events {
  pointer-events: none;
}

.panorama-image {
  width: 100%;
  height: 100%;
  user-select: none;
}
.hotspot-container {
}

.arrow-buttons {
  display: flex;
  justify-content: center;
  position: absolute;
  user-select: none;
  width: 100%;
  background: slategrey;
  opacity: 0.8;
  cursor: default;
}

.arrow-buttons img {
  height: 45px;
  width: 45px;
  cursor: pointer;
}
.arrow-buttons button {
  margin: 0 10px;
  color: #fff;
  border-radius: 5px;
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 5px;
}
.arrow-buttons button:hover {
  scale: 0.9;
}
.arrow-buttons button:active {
  scale: 1.1;
}
.arrow-buttons button:focus {
  outline: none;
}
</style>
