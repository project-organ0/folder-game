<script setup>
defineProps({ modelValue: String, score: Number });
defineEmits(['update:modelValue', 'confirm', 'skip']);
</script>

<template>
  <div class="modal-backdrop">
    <div class="dialog" role="dialog" aria-modal="true">
      <div class="dialog-icon">🗑️</div>
      <div>
        <h2>파일 정리가 완료되었습니다</h2>
        <p>{{ score / 10 }}개 폴더를 정리하고 {{ (score * 12.4).toFixed(1) }}MB의 공간을 확보했습니다.</p>
        <label>작업자 이름</label>
        <input :value="modelValue" placeholder="이름 입력 (선택)" @input="$emit('update:modelValue', $event.target.value)" />
        <div class="actions"><button @click="$emit('confirm')">확인</button><button class="secondary" @click="$emit('skip')">기록 없이 종료</button></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-backdrop { position:fixed; inset:0; z-index:4000; display:grid; place-items:center; background:rgba(0,0,0,.32); }
.dialog { width:min(460px,calc(100vw - 40px)); padding:24px; display:grid; grid-template-columns:52px 1fr; gap:18px; box-sizing:border-box; background:#fff; border:1px solid #888; box-shadow:0 18px 50px rgba(0,0,0,.32); color:#222; text-align:left; }
.dialog-icon { font-size:34px; } h2 { margin:0 0 10px; font-size:17px; font-weight:600; } p { margin:0 0 18px; color:#555; font-size:12px; line-height:1.55; } label { display:block; margin-bottom:5px; font-size:11px; } input { width:100%; height:30px; padding:0 8px; box-sizing:border-box; border:1px solid #888; }
.actions { margin-top:22px; display:flex; justify-content:flex-end; gap:8px; } button { min-width:84px; padding:7px 13px; border:1px solid #0067b8; border-radius:2px; background:#0067b8; color:white; font-size:11px; }.secondary { border-color:#888; background:#f6f6f6; color:#222; }
</style>
