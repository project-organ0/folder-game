<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';
import { ArrowLeft, ArrowRight, ArrowUp, ChevronDown, Search, ShieldCheck, Volume2, VolumeX, X } from 'lucide-vue-next';
import AppleGrid from '@/components/AppleGrid.vue';
import DifficultySelector from '@/components/DifficultySelector.vue';
import GameOverModal from '@/components/GameOverModal.vue';

const difficulties = {
  '한가한 날': { cols: 7, rows: 4, description: '파일 28개' },
  '평범한 날': { cols: 10, rows: 6, description: '파일 60개' },
  '월요일': { cols: 12, rows: 7, description: '파일 84개' },
};

const folderNames = ['회의록_진짜', '예산안_수정', '업무자료', '백업_열지마', '보고서_최종', '기획서_컨펌전', '영수증', '참고자료', '최종_진짜최종', '공유문서', '팀장님요청', '새 폴더 (2)'];
const files = ref([]);
const cols = ref(0);
const rows = ref(0);
const gameStarted = ref(false);
const gameOver = ref(false);
const bossMode = ref(false);
const score = ref(0);
const clearedFiles = ref(0);
const combo = ref(0);
const soundEnabled = ref(true);
const toast = ref(null);
const toastTimer = ref(null);
const timeLeft = ref(60);
const maxTime = 60;
const timer = ref(null);
const username = ref('');
const currentDifficulty = ref('');

const clearedCount = computed(() => clearedFiles.value);
const storageSaved = computed(() => (clearedFiles.value * 12.4).toFixed(1));

const makeBoard = (boardCols, boardRows) => {
  return Array.from({ length: boardCols * boardRows }, (_, index) => ({
    selected: false,
    hidden: false,
    number: Math.ceil(Math.random() * 9),
    label: folderNames[index % folderNames.length],
  }));
};

const selectDifficulty = (level) => {
  currentDifficulty.value = level;
  cols.value = difficulties[level].cols;
  rows.value = difficulties[level].rows;
  files.value = makeBoard(cols.value, rows.value);
  score.value = 0;
  clearedFiles.value = 0;
  combo.value = 0;
  timeLeft.value = maxTime;
  gameStarted.value = true;
  gameOver.value = false;
  startTimer();
};

const playTone = (success) => {
  if (!soundEnabled.value) return;
  const AudioContext = window.AudioContext || window.webkitAudioContext;
  if (!AudioContext) return;
  const context = new AudioContext();
  const oscillator = context.createOscillator();
  const gain = context.createGain();
  oscillator.type = success ? 'sine' : 'square';
  oscillator.frequency.setValueAtTime(success ? 660 : 180, context.currentTime);
  if (success) oscillator.frequency.exponentialRampToValueAtTime(880, context.currentTime + 0.12);
  gain.gain.setValueAtTime(0.06, context.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, context.currentTime + 0.16);
  oscillator.connect(gain).connect(context.destination);
  oscillator.onended = () => context.close();
  oscillator.start();
  oscillator.stop(context.currentTime + 0.17);
};

const showToast = (message, type) => {
  clearTimeout(toastTimer.value);
  toast.value = { message, type };
  toastTimer.value = setTimeout(() => { toast.value = null; }, 900);
};

const handleMatch = ({ valid, sum, count, remaining }) => {
  if (!valid) {
    combo.value = 0;
    playTone(false);
    showToast(`합계 ${sum} · 이 작업을 수행할 수 없습니다`, 'error');
    return;
  }
  combo.value += 1;
  clearedFiles.value += count;
  const earned = count * 10 * combo.value;
  score.value += earned;
  timeLeft.value = Math.min(maxTime, timeLeft.value + 1);
  playTone(true);
  showToast(`${count}개 정리 완료 · +${earned}점${combo.value > 1 ? ` · ${combo.value} COMBO` : ''}`, 'success');
  if (remaining === 0) {
    clearInterval(timer.value);
    setTimeout(() => { gameOver.value = true; }, 450);
  }
};

const startTimer = () => {
  clearInterval(timer.value);
  timer.value = setInterval(() => {
    if (bossMode.value) return;
    timeLeft.value -= 1;
    if (timeLeft.value <= 0) {
      clearInterval(timer.value);
      gameOver.value = true;
    }
  }, 1000);
};

const resetGame = () => {
  clearInterval(timer.value);
  gameStarted.value = false;
  gameOver.value = false;
  files.value = [];
  score.value = 0;
  clearedFiles.value = 0;
  combo.value = 0;
};

const handleKey = (event) => {
  if (event.key === 'Escape') bossMode.value = !bossMode.value;
};

onMounted(() => window.addEventListener('keydown', handleKey));
onBeforeUnmount(() => {
  clearInterval(timer.value);
  clearTimeout(toastTimer.value);
  window.removeEventListener('keydown', handleKey);
});
</script>

<template>
  <main class="desktop-shell">
    <section class="explorer-window">
      <header class="titlebar">
        <div class="title"><span class="mini-folder"></span> 업무 자료</div>
        <div class="window-actions"><button>—</button><button>□</button><button class="close"><X :size="15" /></button></div>
      </header>

      <nav class="toolbar">
        <div class="nav-buttons"><ArrowLeft /><ArrowRight class="muted" /><ArrowUp /></div>
        <div class="address-bar"><span class="mini-folder"></span><span>내 PC</span><span>›</span><strong>업무 자료</strong><ChevronDown :size="14" /></div>
        <div class="search-box"><Search :size="16" /><span>업무 자료 검색</span></div>
      </nav>

      <div class="content-area">
        <aside class="sidebar">
          <div class="side-title">즐겨찾기</div>
          <div class="side-item active"><span>📁</span> 업무 자료</div>
          <div class="side-item"><span>🖥️</span> 바탕 화면</div>
          <div class="side-item"><span>📥</span> 다운로드</div>
          <div class="side-title space">내 PC</div>
          <div class="side-item"><span>💾</span> 로컬 디스크 (C:)</div>
          <div class="boss-hint"><ShieldCheck :size="15" /> ESC 상사 모드</div>
        </aside>

        <section class="main-pane">
          <DifficultySelector v-if="!gameStarted" :difficulties="difficulties" @select="selectDifficulty" />

          <template v-else>
            <div class="command-bar">
              <span><b>✓</b> 이름 숫자의 합이 10인 폴더를 드래그하세요</span>
              <div class="game-stats"><span v-if="combo > 1" class="combo">{{ combo }} COMBO</span><strong>{{ score.toLocaleString() }}점</strong><b>{{ timeLeft }}</b>초</div>
            </div>
            <AppleGrid
              :cols="cols"
              :rows="rows"
              :initial-apples="files"
              :game-over="gameOver"
              @match="handleMatch"
            />
          </template>
        </section>
      </div>

      <footer class="statusbar">
        <span>{{ gameStarted ? `${files.length}개 항목` : '업무 자료 준비됨' }}</span>
        <span v-if="gameStarted">{{ clearedCount }}개 정리됨 · {{ storageSaved }}MB 확보</span>
        <button v-if="gameStarted" class="sound-button" :title="soundEnabled ? '효과음 끄기' : '효과음 켜기'" @click="soundEnabled = !soundEnabled"><Volume2 v-if="soundEnabled" :size="13" /><VolumeX v-else :size="13" /></button>
        <button v-if="gameStarted" @click="resetGame">처음으로</button>
      </footer>
    </section>

    <Transition name="toast">
      <div v-if="toast" :class="['toast', toast.type]">{{ toast.message }}</div>
    </Transition>

    <div v-if="bossMode" class="boss-overlay" @click="bossMode = false">
      <div class="sheet-title">2026년 3분기 운영비 정산 현황</div>
      <div class="formula">fx　=SUM(C4:C18)</div>
      <table>
        <thead><tr><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th></tr></thead>
        <tbody><tr v-for="row in 16" :key="row"><th>{{ row }}</th><td>{{ row > 2 ? ['인건비','마케팅','운영비','개발비'][row % 4] : '' }}</td><td>{{ row > 2 ? `2026-0${(row % 6) + 1}` : '' }}</td><td>{{ row > 2 ? (row * 137000).toLocaleString() : '' }}</td><td></td><td></td></tr></tbody>
      </table>
      <div class="overlay-tip">ESC를 누르거나 화면을 클릭해 돌아가기</div>
    </div>

    <GameOverModal v-if="gameOver" v-model="username" :score="score" :cleared-count="clearedCount" :completed="timeLeft > 0" @confirm="resetGame" @skip="resetGame" />
  </main>
</template>

<style scoped>
.desktop-shell { min-height: 100vh; padding: 28px; box-sizing: border-box; background: linear-gradient(135deg, #1469b8, #68b2e3); color: #1f1f1f; }
.explorer-window { max-width: 1180px; min-height: calc(100vh - 56px); margin: auto; display: flex; flex-direction: column; overflow: hidden; background: #fff; border: 1px solid #777; box-shadow: 0 20px 55px rgba(0,0,0,.3); }
.titlebar { height: 36px; display: flex; align-items: center; justify-content: space-between; padding-left: 12px; background: #f7f7f7; }
.title { display: flex; align-items: center; gap: 8px; font-size: 12px; }
.mini-folder { display: inline-block; width: 16px; height: 12px; border-radius: 2px; background: #f1bd3e; }
.window-actions { height: 100%; display: flex; }
.window-actions button { width: 46px; border: 0; border-radius: 0; background: transparent; color: #222; padding: 0; display: grid; place-items: center; }
.window-actions button:hover { background: #e5e5e5; }.window-actions .close:hover { background: #e81123; color: white; }
.toolbar { min-height: 50px; display: flex; gap: 10px; align-items: center; padding: 0 12px; border-block: 1px solid #ddd; }
.nav-buttons { display: flex; gap: 15px; color: #444; }.nav-buttons svg { width: 18px; }.nav-buttons .muted { color: #aaa; }
.address-bar,.search-box { height: 32px; border: 1px solid #aaa; display: flex; align-items: center; gap: 8px; padding: 0 10px; font-size: 12px; }
.address-bar { flex: 1; }.address-bar svg { margin-left: auto; }.search-box { width: 210px; color: #777; }
.content-area { flex: 1; display: flex; min-height: 0; }
.sidebar { width: 190px; flex: 0 0 190px; padding: 18px 8px; border-right: 1px solid #ddd; background: #fafafa; box-sizing: border-box; }
.side-title { padding: 0 10px 8px; font-size: 11px; color: #666; text-transform: uppercase; }.side-title.space { margin-top: 18px; }
.side-item { padding: 8px 10px; font-size: 12px; border-radius: 3px; }.side-item span { margin-right: 7px; }.side-item.active { background: #dbeaf8; }
.boss-hint { margin: 28px 8px 0; padding: 10px; display: flex; align-items: center; gap: 6px; border: 1px solid #ddd; color: #555; font-size: 11px; background: white; }
.main-pane { flex: 1; overflow: auto; position: relative; }
.command-bar { position: sticky; top: 0; z-index: 10; min-height: 42px; padding: 0 18px; display: flex; align-items: center; justify-content: space-between; border-bottom: 1px solid #e5e5e5; background: rgba(255,255,255,.96); font-size: 12px; }.command-bar b { color: #0067b8; }
.game-stats { display:flex; align-items:center; gap:12px; }.game-stats strong { font-size:12px; }.combo { padding:3px 7px; border-radius:10px; background:#fff1b8; color:#8a5700; font-size:10px; font-weight:700; animation:combo-pop .2s ease; }
.statusbar { min-height: 30px; padding: 0 12px; display: flex; align-items: center; gap: 18px; border-top: 1px solid #ddd; background: #f7f7f7; font-size: 11px; }.statusbar button { padding: 2px 9px; font-size: 11px; }.statusbar button:not(.sound-button) { margin-left:0; }.statusbar span:nth-child(2) { margin-right:auto; }.sound-button { display:grid; place-items:center; padding:3px 6px !important; border:0; background:transparent; color:#444; }
.toast { position:fixed; z-index:3000; left:50%; top:92px; transform:translateX(-50%); padding:10px 16px; border:1px solid; border-radius:3px; box-shadow:0 8px 22px rgba(0,0,0,.2); background:#fff; font-size:12px; font-weight:600; }.toast.success { border-color:#18864b; color:#116c3a; background:#f2fff7; }.toast.error { border-color:#c42b1c; color:#a4261c; background:#fff5f3; }
.toast-enter-active,.toast-leave-active { transition:all .18s ease; }.toast-enter-from,.toast-leave-to { opacity:0; transform:translate(-50%,-8px); }
@keyframes combo-pop { from { transform:scale(.7); } to { transform:scale(1); } }
.boss-overlay { position: fixed; inset: 0; z-index: 5000; padding: 20px; background: #f3f3f3; font-family: Calibri, Arial, sans-serif; overflow: hidden; }
.sheet-title { padding: 13px; background: #217346; color: white; font-size: 18px; }.formula { margin: 10px 0; padding: 8px; border: 1px solid #bbb; background: white; }
table { width: 100%; border-collapse: collapse; background: white; font-size: 12px; } th,td { height: 24px; border: 1px solid #d5d5d5; text-align: left; padding: 0 7px; } thead th,tbody th { background: #eee; text-align: center; font-weight: normal; }
.overlay-tip { position: fixed; right: 18px; bottom: 14px; padding: 8px 12px; border-radius: 4px; background: #222; color: white; font-size: 11px; opacity: .8; }
@media (max-width: 760px) { .desktop-shell { padding: 0; }.explorer-window { min-height: 100vh; border: 0; }.sidebar { display:none; }.search-box { display:none; }.command-bar { align-items:flex-start; flex-direction:column; gap:5px; padding-block:8px; }.game-stats { width:100%; justify-content:space-between; }.window-actions button { width: 38px; }.toast { top:105px; width:max-content; max-width:calc(100vw - 30px); } }
</style>
