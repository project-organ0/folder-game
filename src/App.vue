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
      content: '폴더를 드래그해 정리하는 숫자 퍼즐 게임, 퇴근 전 파일정리.',
    },
    {
      name: 'keywords',
      content: '폴더 게임, 숫자 게임, 10 만들기, 퍼즐 게임, 파일 정리 게임',
    },
    { property: 'og:title', content: '퇴근 전 파일정리' },
    {
      property: 'og:description',
      content: '업무 중인 척 폴더를 정리하는 숫자 퍼즐 게임.',
    },
    { property: 'og:type', content: 'website' },
    { property: 'og:url', content: 'https://github.com/project-organ0/folder-game' },
    { name: 'twitter:card', content: 'summary_large_image' },
  ],
  link: [{ rel: 'icon', href: '/folder.svg', type: 'image/svg+xml' }],
});
</script>
