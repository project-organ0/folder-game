<script setup>
defineProps({ difficulties: Object });
defineEmits(['select']);
</script>

<template>
  <div class="welcome">
    <div class="hero-folder"><span></span></div>
    <p class="eyebrow">QUICK ACCESS</p>
    <h1>퇴근 전 파일정리</h1>
    <p class="description">폴더명 끝 숫자의 합이 <b>10</b>이 되도록 드래그해서<br />회사 PC를 말끔하게 정리하세요.</p>
    <div class="levels">
      <button v-for="(info, level) in difficulties" :key="level" @click="$emit('select', level)">
        <span class="level-name">{{ level }}</span>
        <span class="level-info">{{ info.description }} 정리하기</span>
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
.levels { max-width: 560px; margin: auto; display: grid; grid-template-columns: repeat(3,1fr); gap: 10px; }
.levels button { padding: 15px 12px; display:flex; flex-direction:column; gap:5px; border:1px solid #bbb; border-radius:2px; background:#fff; color:#222; }
.levels button:hover { border-color:#0078d4; background:#f2f8fd; }.level-name { font-size:14px; font-weight:600; }.level-info { color:#777; font-size:10px; }
.tip { margin-top:28px; color:#777; font-size:11px; } kbd { padding:2px 6px; border:1px solid #aaa; border-bottom-width:2px; border-radius:3px; background:#fafafa; color:#333; }
@media (max-width:600px) { .levels { grid-template-columns:1fr; max-width:300px; }.welcome { padding-top:35px; } }
</style>
