<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue';
import Apple from '@/components/Apple.vue';

const props = defineProps({
  cols: Number,
  rows: Number,
  initialApples: Array,
  gameOver: Boolean,
});

const emit = defineEmits(['match', 'selection-change']);
const apples = ref([]);
const appleRefs = ref([]);
const selecting = ref(false);
const dragBoxStyle = ref({ display: 'none' });
const start = ref({ x: 0, y: 0 });
const selectionSum = ref(0);
const selectionCount = ref(0);

watch(() => props.initialApples, (items) => {
  apples.value = [...items];
  appleRefs.value = Array(items.length).fill(null);
}, { immediate: true });

const setAppleRef = (el, index) => {
  if (el) appleRefs.value[index] = el;
};

const onPointerDown = (event) => {
  if (props.gameOver || event.button !== 0) return;
  event.preventDefault();
  start.value = { x: event.clientX, y: event.clientY };
  selecting.value = true;
  apples.value.forEach((item) => { item.selected = false; });
  selectionSum.value = 0;
  selectionCount.value = 0;
};

const onPointerMove = (event) => {
  if (!selecting.value || props.gameOver) return;
  const x = Math.min(start.value.x, event.clientX);
  const y = Math.min(start.value.y, event.clientY);
  const width = Math.abs(start.value.x - event.clientX);
  const height = Math.abs(start.value.y - event.clientY);

  dragBoxStyle.value = {
    display: 'block', left: `${x}px`, top: `${y}px`,
    width: `${width}px`, height: `${height}px`,
  };

  appleRefs.value.forEach((el, index) => {
    if (!el || apples.value[index].hidden) return;
    const rect = el.getBoundingClientRect();
    const cx = rect.left + rect.width / 2;
    const cy = rect.top + rect.height / 2;
    apples.value[index].selected = cx >= x && cx <= x + width && cy >= y && cy <= y + height;
  });

  const selected = apples.value.filter((item) => item.selected && !item.hidden);
  selectionSum.value = selected.reduce((total, item) => total + item.number, 0);
  selectionCount.value = selected.length;
  emit('selection-change', { sum: selectionSum.value, count: selectionCount.value });
};

const onPointerUp = () => {
  if (!selecting.value) return;
  selecting.value = false;
  dragBoxStyle.value = { display: 'none' };

  const selected = apples.value.filter((item) => item.selected && !item.hidden);
  const sum = selected.reduce((total, item) => total + item.number, 0);
  if (sum === 10) {
    selected.forEach((item) => { item.hidden = true; });
  }
  if (selected.length) {
    emit('match', {
      valid: sum === 10,
      sum,
      count: selected.length,
      remaining: apples.value.filter((item) => !item.hidden).length,
    });
  }
  apples.value.forEach((item) => { item.selected = false; });
  selectionSum.value = 0;
  selectionCount.value = 0;
  emit('selection-change', { sum: 0, count: 0 });
};

onMounted(() => {
  window.addEventListener('pointerup', onPointerUp);
  window.addEventListener('pointermove', onPointerMove);
});

onBeforeUnmount(() => {
  window.removeEventListener('pointerup', onPointerUp);
  window.removeEventListener('pointermove', onPointerMove);
});
</script>

<template>
  <div class="file-grid-wrapper" @pointerdown="onPointerDown">
    <div class="drag-box" :style="dragBoxStyle"></div>
    <div v-if="selecting && selectionCount" :class="['sum-badge', { ready: selectionSum === 10, over: selectionSum > 10 }]">
      {{ selectionSum }} <small>/ 10</small>
    </div>
    <div class="file-grid" :style="{ gridTemplateColumns: `repeat(${cols}, 82px)` }">
      <div
        v-for="(apple, index) in apples"
        :key="index"
        :ref="(el) => setAppleRef(el, index)"
        class="file-wrapper"
      >
        <Apple v-bind="apple" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.file-grid-wrapper { position: relative; width: 100%; min-height: 360px; }
.file-grid { display: grid; gap: 8px 4px; align-content: start; padding: 18px; }
.file-wrapper { width: 82px; height: 76px; }
.drag-box {
  position: fixed;
  border: 1px solid #0078d7;
  background: rgba(0, 120, 215, 0.12);
  z-index: 1000;
  pointer-events: none;
}
.sum-badge { position:fixed; z-index:1001; right:24px; bottom:48px; min-width:72px; padding:9px 13px; border:1px solid #777; border-radius:3px; background:#fff; box-shadow:0 5px 18px rgba(0,0,0,.2); color:#333; font-size:18px; font-weight:700; text-align:center; pointer-events:none; }
.sum-badge small { color:#888; font-size:11px; font-weight:400; }.sum-badge.ready { border-color:#168342; color:#168342; background:#f1fff5; }.sum-badge.over { border-color:#c42b1c; color:#c42b1c; background:#fff4f2; }
@media (max-width: 760px) {
  .file-grid { grid-template-columns: repeat(3, 82px) !important; justify-content: center; padding-inline: 8px; }
}
</style>
