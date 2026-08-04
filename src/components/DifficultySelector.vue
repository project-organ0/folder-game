<script setup>
defineProps({ difficulties: Object });
defineEmits(['select']);
</script>

<template>
  <div class="welcome">
    <div class="hero-folder"><span></span></div>
    <p class="eyebrow">QUICK ACCESS</p>
    <h1>퇴근 전 파일정리</h1>
    <p class="description">같은 프로젝트의 <b>기획서 · 시안 · 견적서</b>를 찾아<br />하나의 업무 세트로 정리하세요.</p>
    <div class="start-action">
      <button v-for="(info, level) in difficulties" :key="level" @click="$emit('select', level)">
        <span class="level-name">파일 정리 시작</span>
        <span class="level-info">한 화면 집중 모드 · 제한시간 60초</span>
      </button>
    </div>
    <p class="tip">TIP · 누가 오면 <kbd>ESC</kbd>를 누르세요</p>
  </div>
</template>

<style scoped>
.welcome { min-height: 520px; padding: 50px 24px 30px; text-align: center; box-sizing: border-box; }
.hero-folder { position: relative; width: 80px; height: 58px; margin: 0 auto 20px; border-radius: 6px; background: linear-gradient(145deg,#ffd76a,#eba824); box-shadow: 0 10px 22px rgba(181,126,0,.22); }
.hero-folder span { position:absolute; width:35px; height:13px; left:5px; top:-8px; border-radius:5px 5px 0 0; background:#e2a326; }
.eyebrow { margin: 0 0 8px; color: #0078d4; font-size: 10px; font-weight: 700; letter-spacing: .18em; }
h1 { margin: 0; color: #202020; font-size: 30px; font-weight: 600; }
.description { margin: 14px 0 30px; color: #666; font-size: 14px; line-height: 1.7; }.description b { color:#0078d4; }
.start-action { max-width: 320px; margin: auto; }
.start-action button { width:100%; padding:16px 20px; display:flex; align-items:center; justify-content:space-between; gap:14px; border:1px solid #0067b8; border-radius:3px; background:#0078d4; color:#fff; box-shadow:0 5px 14px rgba(0,103,184,.2); }
.start-action button:hover { background:#006cbe; }.start-action button:active { transform:translateY(1px); }.level-name { font-size:14px; font-weight:700; }.level-info { color:rgba(255,255,255,.78); font-size:10px; }
.tip { margin-top:28px; color:#777; font-size:11px; } kbd { padding:2px 6px; border:1px solid #aaa; border-bottom-width:2px; border-radius:3px; background:#fafafa; color:#333; }
@media (max-width:600px) { .start-action { max-width:300px; }.welcome { padding-top:35px; } }
</style>
