<template>
  <RouterView />
</template>

<script setup>
import { RouterView } from 'vue-router';
import { useHead } from '@vueuse/head';

useHead({
  title: '업무 자료 - 파일 탐색기',
  meta: [
    {
      name: 'description',
      content: '같은 프로젝트의 업무 파일을 찾아 묶는 드래그 퍼즐 게임, 퇴근 전 파일정리.',
    },
    {
      name: 'keywords',
      content: '파일 정리 게임, 업무 퍼즐, 드래그 퍼즐, 프로젝트 분류 게임',
    },
    { property: 'og:title', content: '퇴근 전 파일정리' },
    {
      property: 'og:description',
      content: '업무 중인 척 프로젝트별 파일 세트를 정리하는 퍼즐 게임.',
    },
    { property: 'og:type', content: 'website' },
    { property: 'og:url', content: 'https://github.com/project-organ0/folder-game' },
    { name: 'twitter:card', content: 'summary_large_image' },
  ],
  link: [{ rel: 'icon', href: '/folder.svg', type: 'image/svg+xml' }],
});
</script>
