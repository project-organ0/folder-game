<script setup>
import { ref, watch, onBeforeUnmount } from 'vue';
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
const selectionCount = ref(0);
const selectionProject = ref('');
const selectionReady = ref(false);
const mergeEffect = ref(null);
const mergeTimer = ref(null);
const selectionTimer = ref(null);
const resolving = ref(false);

watch(() => props.initialApples, (items) => {
  apples.value = [...items];
  appleRefs.value = Array(items.length).fill(null);
}, { immediate: true });

const setAppleRef = (el, index) => {
  if (el) appleRefs.value[index] = el;
};

const updateSelection = () => {
  const selected = apples.value.filter((item) => item.selected && !item.hidden);
  selectionCount.value = selected.length;
  const projects = new Set(selected.map((item) => item.project));
  const types = new Set(selected.map((item) => item.type));
  selectionProject.value = projects.size === 1 ? selected[0]?.project ?? '' : '';
  selectionReady.value = selected.length === 3 && projects.size === 1 && types.size === 3;
  emit('selection-change', { project: selectionProject.value, count: selectionCount.value, ready: selectionReady.value });
};

const resolveSelection = () => {
  const selected = apples.value.filter((item) => item.selected && !item.hidden);
  const projects = new Set(selected.map((item) => item.project));
  const types = new Set(selected.map((item) => item.type));
  const valid = selected.length === 3 && projects.size === 1 && types.size === 3;
  if (valid) {
    const selectedIndexes = apples.value
      .map((item, index) => (item.selected && !item.hidden ? index : -1))
      .filter((index) => index >= 0);
    const rects = selectedIndexes.map((index) => appleRefs.value[index].getBoundingClientRect());
    const center = rects.reduce((point, rect) => ({
      x: point.x + rect.left + rect.width / 2,
      y: point.y + rect.top + rect.height / 2,
    }), { x: 0, y: 0 });
    center.x /= rects.length;
    center.y /= rects.length;
    selectedIndexes.forEach((index, itemIndex) => {
      const rect = rects[itemIndex];
      appleRefs.value[index].style.setProperty('--merge-x', `${center.x - rect.left - rect.width / 2}px`);
      appleRefs.value[index].style.setProperty('--merge-y', `${center.y - rect.top - rect.height / 2}px`);
      appleRefs.value[index].style.setProperty('--merge-delay', `${itemIndex * 35}ms`);
    });
    clearTimeout(mergeTimer.value);
    mergeEffect.value = { x: center.x, y: center.y, project: selected[0].project };
    mergeTimer.value = setTimeout(() => { mergeEffect.value = null; }, 720);
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
  resolving.value = false;
  emit('selection-change', { project: '', count: 0, ready: false });
};

const toggleFile = (index) => {
  const item = apples.value[index];
  if (props.gameOver || resolving.value || !item || item.hidden) return;
  item.selected = !item.selected;
  updateSelection();
  if (selectionCount.value === 3) {
    resolving.value = true;
    clearTimeout(selectionTimer.value);
    selectionTimer.value = setTimeout(resolveSelection, 150);
  }
};

onBeforeUnmount(() => {
  clearTimeout(mergeTimer.value);
  clearTimeout(selectionTimer.value);
});
</script>

<template>
  <div class="file-grid-wrapper">
    <div v-if="selectionCount" :class="['set-badge', { ready: selectionReady, mixed: !selectionProject }]">
      {{ selectionProject || '서로 다른 프로젝트' }} <small>{{ selectionCount }} / 3 선택</small>
    </div>
    <div
      v-if="mergeEffect"
      class="merge-effect"
      :style="{ left: `${mergeEffect.x}px`, top: `${mergeEffect.y}px` }"
      aria-hidden="true"
    >
      <span class="merge-ring"></span>
      <span class="merged-folder"><i></i><b>✓</b></span>
      <strong>{{ mergeEffect.project }} 정리 완료</strong>
    </div>
    <div class="file-grid" :style="{ gridTemplateColumns: `repeat(${cols}, minmax(0, 1fr))`, gridTemplateRows: `repeat(${rows}, minmax(0, 1fr))` }">
      <div
        v-for="(apple, index) in apples"
        :key="index"
        :ref="(el) => setAppleRef(el, index)"
        :class="['file-wrapper', { unavailable: apple.hidden }]"
        role="button"
        :tabindex="apple.hidden ? -1 : 0"
        :aria-label="`${apple.project} ${apple.label} ${apple.selected ? '선택 해제' : '선택'}`"
        @click="toggleFile(index)"
        @keydown.enter.prevent="toggleFile(index)"
        @keydown.space.prevent="toggleFile(index)"
      >
        <Apple v-bind="apple" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.file-grid-wrapper { position:relative; width:100%; min-height:0; flex:1; overflow:hidden; user-select:none; -webkit-user-select:none; }
.file-grid { width:100%; height:100%; display:grid; gap:4px; padding:12px; }
.file-wrapper { min-width:0; min-height:0; cursor:pointer; border-radius:4px; outline:none; -webkit-tap-highlight-color:transparent; }
.file-wrapper:hover { background:#f4f8fc; }
.file-wrapper:focus-visible { box-shadow:inset 0 0 0 2px #0078d4; }
.file-wrapper.unavailable { pointer-events:none; }
.set-badge { position:fixed; z-index:1001; right:24px; bottom:48px; min-width:110px; padding:9px 13px; border:1px solid #777; border-radius:3px; background:#fff; box-shadow:0 5px 18px rgba(0,0,0,.2); color:#333; font-size:14px; font-weight:700; text-align:center; pointer-events:none; }
.set-badge small { display:block; margin-top:3px; color:#888; font-size:10px; font-weight:400; }.set-badge.ready { border-color:#168342; color:#168342; background:#f1fff5; }.set-badge.mixed { border-color:#c42b1c; color:#c42b1c; background:#fff4f2; }
.merge-effect { position:fixed; z-index:1500; width:170px; height:130px; transform:translate(-50%,-50%); display:flex; flex-direction:column; align-items:center; pointer-events:none; }
.merge-ring { position:absolute; left:50%; top:38px; width:52px; height:52px; border:2px solid #168342; border-radius:50%; transform:translate(-50%,-50%); animation:merge-ring .62s ease-out forwards; }
.merged-folder { position:relative; width:58px; height:43px; margin-top:16px; border-radius:5px; background:linear-gradient(145deg,#ffd568,#e7a724); box-shadow:0 7px 18px rgba(141,95,0,.28); animation:folder-pop .62s cubic-bezier(.16,1.35,.36,1) both; }
.merged-folder i { position:absolute; left:5px; top:-8px; width:27px; height:11px; border-radius:4px 4px 0 0; background:#d99b1d; }
.merged-folder b { position:absolute; right:-8px; bottom:-7px; width:25px; height:25px; display:grid; place-items:center; border:3px solid white; border-radius:50%; background:#168342; color:white; font-size:14px; }
.merge-effect strong { margin-top:13px; padding:5px 9px; border-radius:12px; background:#168342; box-shadow:0 4px 12px rgba(0,0,0,.18); color:white; font-size:10px; white-space:nowrap; animation:merge-label .6s .12s ease both; }
@keyframes merge-ring { from { opacity:.9; transform:translate(-50%,-50%) scale(.2); } to { opacity:0; transform:translate(-50%,-50%) scale(2.6); } }
@keyframes folder-pop { 0% { opacity:0; transform:scale(.2) rotate(-8deg); } 58% { opacity:1; transform:scale(1.14) rotate(2deg); } 100% { opacity:1; transform:scale(1) rotate(0); } }
@keyframes merge-label { from { opacity:0; transform:translateY(7px); } to { opacity:1; transform:translateY(0); } }
@media (max-width: 760px) {
  .file-grid { gap:2px; padding:6px 4px; }
}
</style>
