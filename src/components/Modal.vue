<template>
  <div class="modal-overlay">
    <div
      class="modal"
      :style="{ maxWidth: `${maxWidth}px` }"
      :class="{ 'full-height': fullHeight, 'full-width': fullWidth }"
    >
      <div class="modal_header">
        <div class="modal_title">
          {{ title }}
        </div>
        <button type="button" class="modal_close" @click="$emit('close')">
          <font-awesome-icon icon="fa-regular fa-circle-xmark" />
        </button>
      </div>
      <div class="modal_body">
        <slot />
      </div>
    </div>
  </div>
</template>
<script setup>
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

const props = defineProps({
  title: String,
  maxWidth: {
    type: Number,
    default: 600,
  },
  fullWidth: {
    type: Boolean,
    default: false,
  },
  fullHeight: {
    type: Boolean,
    default: false,
  },
});
</script>
<style lang="scss">
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  z-index: 1300;
  width: 100%;
  max-height: 100vh;
  max-width: 100vw;
  background-color: rgb(255, 255, 255);
  border-radius: 5px;
  padding: 10px 20px;

  &.full-height {
    height: 92vh;
    margin: 1.5rem auto;
    max-height: 96vh !important;
  }

  &.full-width {
    width: 100vw;
    max-width: 96vw !important;
  }
}

.modal_header {
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
}

.modal_title {
  font-weight: bold;
}

.modal_body {
  padding-right: 20px;
  height: 100%;
  overflow: auto;
}

.modal_close {
  font-size: 1.6rem;
  color: rgba(0, 0, 0, 0.3);
}

.modal_close svg {
  display: block;
}

.modal_close:hover {
  color: rgba(0, 0, 0, 0.6);
}
</style>
