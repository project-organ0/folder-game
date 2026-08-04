<template>
  <div
    v-show="visible"
    :class="['file-item', { selected, deleting: shouldAnimateHide }]"
    @animationend="onAnimationEnd"
  >
    <div :class="['file-icon', `type-${type}`]" aria-hidden="true">
      <span class="file-icon__fold"></span>
      <span class="file-icon__preview">
        <i v-if="type === 'brief'" class="preview-lines"></i>
        <i v-else-if="type === 'asset'" class="preview-image"></i>
        <i v-else class="preview-sheet"></i>
      </span>
      <span class="file-icon__badge">{{ type === 'brief' ? 'W' : type === 'asset' ? 'P' : 'X' }}</span>
      <span class="file-icon__extension">{{ extension }}</span>
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
  width: 42px;
  height: 44px;
  box-sizing: border-box;
  border: 1px solid #c4c7cb;
  border-radius: 3px;
  background: linear-gradient(145deg, #fff 0%, #f7f8fa 100%);
  box-shadow: 0 1px 2px rgba(0, 0, 0, .16), 0 3px 7px rgba(0, 0, 0, .08);
  transition: transform .12s ease, box-shadow .12s ease;
  overflow: visible;
}
.file-icon__fold {
  position: absolute;
  top: -1px;
  right: -1px;
  width: 12px;
  height: 12px;
  border-left: 1px solid #c4c7cb;
  border-bottom: 1px solid #c4c7cb;
  border-radius: 0 3px 0 2px;
  background: linear-gradient(225deg, #eef0f3 0 48%, #fff 50%);
  clip-path: polygon(0 0, 100% 100%, 0 100%);
}
.file-icon__preview {
  position: absolute;
  inset: 8px 7px 10px;
  overflow: hidden;
}
.file-icon__preview i { display: block; width: 100%; height: 100%; }
.preview-lines {
  background: repeating-linear-gradient(to bottom, #d5d9df 0 2px, transparent 2px 5px);
}
.preview-image {
  border-radius: 2px;
  background:
    radial-gradient(circle at 72% 28%, #f1c95b 0 2px, transparent 2.5px),
    linear-gradient(145deg, transparent 46%, #b9d38d 47% 66%, transparent 67%),
    linear-gradient(35deg, transparent 38%, #79b7a3 39% 68%, transparent 69%),
    #e8f2f0;
}
.preview-sheet {
  border: 1px solid #d8e6dc;
  background:
    repeating-linear-gradient(to right, transparent 0 6px, #d8e6dc 6px 7px),
    repeating-linear-gradient(to bottom, transparent 0 5px, #d8e6dc 5px 6px),
    #f4faf6;
}
.file-icon__badge {
  position: absolute;
  left: -5px;
  bottom: 3px;
  width: 18px;
  height: 18px;
  display: grid;
  place-items: center;
  border-radius: 2px;
  background: #2b579a;
  box-shadow: 0 1px 2px rgba(0, 0, 0, .22);
  color: white;
  font: 700 11px/1 "Segoe UI", sans-serif;
}
.type-asset .file-icon__badge { background: #9a4caf; }
.type-budget .file-icon__badge { background: #217346; }
.file-icon__extension {
  position: absolute;
  right: 3px;
  bottom: 2px;
  color: #697079;
  font-size: 6px;
  font-weight: 700;
  letter-spacing: -.2px;
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
  z-index: 1200;
  animation: recycle 0.46s var(--merge-delay, 0ms) cubic-bezier(.55, 0, .2, 1) forwards;
}

@keyframes recycle {
  0% { transform:translate(0,0) scale(1) rotate(0); opacity:1; filter:brightness(1); }
  72% { opacity:1; filter:brightness(1.15); }
  100% { transform:translate(var(--merge-x, 0),var(--merge-y, 36px)) scale(.08) rotate(16deg); opacity:0; filter:brightness(1.3); }
}

@media (max-width: 760px) {
  .file-item { gap:3px; padding:3px 1px 2px; }
  .file-icon { width:36px; height:38px; }
  .file-icon__preview { inset:7px 6px 9px; }
  .file-icon__badge { left:-3px; width:16px; height:16px; font-size:9px; }
  .file-icon__extension { font-size:5px; }
  .file-name { max-width:58px; min-height:20px; font-size:8px; line-height:10px; }
}
</style>
