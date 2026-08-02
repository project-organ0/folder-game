<template>
  <div
    v-show="visible"
    :class="['file-item', { selected, deleting: shouldAnimateHide }]"
    @animationend="onAnimationEnd"
  >
    <div class="folder-icon" aria-hidden="true">
      <div class="folder-tab"></div>
      <div class="folder-body"></div>
    </div>
    <span class="file-name">{{ label }}_{{ number }}</span>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  selected: Boolean,
  hidden: Boolean,
  number: Number,
  label: { type: String, default: '업무자료' },
});

const visible = ref(true);
const shouldAnimateHide = ref(false);

watch(() => props.hidden, (hidden) => {
  if (hidden) shouldAnimateHide.value = true;
});

const onAnimationEnd = () => {
  if (shouldAnimateHide.value) visible.value = false;
};
</script>

<style scoped>
.file-item {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  gap: 5px;
  padding: 6px 4px 4px;
  border: 1px solid transparent;
  border-radius: 2px;
  box-sizing: border-box;
  color: #1f1f1f;
}

.file-item.selected {
  background: rgba(0, 120, 215, 0.16);
  border-color: rgba(0, 120, 215, 0.48);
}
.file-item.selected .folder-icon { transform:translateY(-2px); }

.folder-icon {
  position: relative;
  width: 42px;
  height: 34px;
  filter: drop-shadow(0 1px 1px rgba(0, 0, 0, 0.18));
  transition:transform .12s ease;
}

.folder-tab {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 19px;
  height: 9px;
  background: #e6aa25;
  border-radius: 3px 3px 0 0;
}

.folder-body {
  position: absolute;
  inset: 8px 1px 1px;
  border-radius: 3px;
  background: linear-gradient(145deg, #ffd76a 0%, #f2b934 68%, #df9e1d 100%);
  border: 1px solid #d99b22;
}

.folder-body::after {
  content: '';
  position: absolute;
  inset: 4px 3px 3px;
  border-radius: 2px;
  border-top: 1px solid rgba(255, 255, 255, 0.7);
}

.file-name {
  max-width: 74px;
  min-height: 26px;
  overflow: hidden;
  font-size: 11px;
  line-height: 13px;
  text-align: center;
  overflow-wrap: anywhere;
}

.file-item.deleting {
  animation: recycle 0.38s cubic-bezier(.4, 0, .2, 1) forwards;
}

@keyframes recycle {
  to { transform: scale(0.2) translateY(40px); opacity: 0; }
}
</style>
