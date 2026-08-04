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
const selectionCount = ref(0);
const selectionProject = ref('');
const selectionReady = ref(false);

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
  selectionCount.value = 0;
  selectionProject.value = '';
  selectionReady.value = false;
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
  selectionCount.value = selected.length;
  const projects = new Set(selected.map((item) => item.project));
  const types = new Set(selected.map((item) => item.type));
  selectionProject.value = projects.size === 1 ? selected[0]?.project ?? '' : '';
  selectionReady.value = selected.length === 3 && projects.size === 1 && types.size === 3;
  emit('selection-change', { project: selectionProject.value, count: selectionCount.value, ready: selectionReady.value });
};

const onPointerUp = () => {
  if (!selecting.value) return;
  selecting.value = false;
  dragBoxStyle.value = { display: 'none' };

  const selected = apples.value.filter((item) => item.selected && !item.hidden);
  const projects = new Set(selected.map((item) => item.project));
  const types = new Set(selected.map((item) => item.type));
  const valid = selected.length === 3 && projects.size === 1 && types.size === 3;
  if (valid) {
    selected.forEach((item) => { item.hidden = true; });
  }
  if (selected.length) {
    emit('match', {
      valid,
      reason: projects.size > 1 ? 'mixed' : 'incomplete',
      project: projects.size === 1 ? selected[0].project : null,
      count: selected.length,
      remaining: apples.value.filter((item) => !item.hidden).length,
    });
  }
  apples.value.forEach((item) => { item.selected = false; });
  selectionCount.value = 0;
  selectionProject.value = '';
  selectionReady.value = false;
  emit('selection-change', { project: '', count: 0, ready: false });
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
    <div v-if="selecting && selectionCount" :class="['set-badge', { ready: selectionReady, mixed: !selectionProject }]">
      {{ selectionProject || '프로젝트 혼합' }} <small>{{ selectionCount }} / 3</small>
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
.file-grid-wrapper { position: relative; width: 100%; min-height: 360px; touch-action:none; user-select:none; -webkit-user-select:none; }
.file-grid { display: grid; gap: 8px 4px; align-content: start; padding: 18px; }
.file-wrapper { width: 82px; height: 76px; }
.drag-box {
  position: fixed;
  border: 1px solid #0078d7;
  background: rgba(0, 120, 215, 0.12);
  z-index: 1000;
  pointer-events: none;
}
.set-badge { position:fixed; z-index:1001; right:24px; bottom:48px; min-width:110px; padding:9px 13px; border:1px solid #777; border-radius:3px; background:#fff; box-shadow:0 5px 18px rgba(0,0,0,.2); color:#333; font-size:14px; font-weight:700; text-align:center; pointer-events:none; }
.set-badge small { display:block; margin-top:3px; color:#888; font-size:10px; font-weight:400; }.set-badge.ready { border-color:#168342; color:#168342; background:#f1fff5; }.set-badge.mixed { border-color:#c42b1c; color:#c42b1c; background:#fff4f2; }
@media (max-width: 760px) {
  .file-grid { grid-template-columns: repeat(3, 82px) !important; justify-content: center; padding-inline: 8px; }
}
</style>
