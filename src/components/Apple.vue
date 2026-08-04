<template>
  <div
    v-show="visible"
    :class="['file-item', { selected, deleting: shouldAnimateHide }]"
    @animationend="onAnimationEnd"
  >
    <div :class="['file-icon', `type-${type}`]" aria-hidden="true">
      <span>{{ extension }}</span>
    </div>
    <span class="file-name">{{ project }}_{{ label }}_{{ suffix }}</span>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  selected: Boolean,
  hidden: Boolean,
  project: { type: String, default: 'PROJECT' },
  type: { type: String, default: 'brief' },
  label: { type: String, default: '업무자료' },
  extension: { type: String, default: 'FILE' },
  suffix: { type: String, default: '최종' },
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
.file-item.selected .file-icon { transform:translateY(-2px); }

.file-icon {
  position: relative;
  width: 36px;
  height: 38px;
  display:flex;
  align-items:flex-end;
  justify-content:center;
  padding-bottom:5px;
  box-sizing:border-box;
  border:1px solid #a8a8a8;
  border-radius:2px;
  background:linear-gradient(135deg,#fff 0 72%,#e8e8e8 73%);
  filter: drop-shadow(0 1px 1px rgba(0, 0, 0, 0.18));
  transition:transform .12s ease;
}
.file-icon:before {
  content:'';
  position:absolute;
  left:4px;
  right:4px;
  top:7px;
  height:13px;
  border-radius:1px;
  background:#3f74b5;
}
.file-icon.type-asset:before { background:#a55ac7; }
.file-icon.type-budget:before { background:#36a067; }
.file-icon span {
  position:relative;
  z-index:1;
  font-size:7px;
  font-weight:800;
  color:#555;
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
