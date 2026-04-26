<script setup lang="ts">
import type { Member } from '../types';
import MemberCard from './MemberCard.vue';

defineProps<{
  members: Member[];
}>();
</script>

<template>
  <div class="max-w-400 mx-auto w-full px-6 py-12 md:py-20 lg:py-24">
    <header class="flex flex-col items-center mb-16 text-center w-full max-w-4xl mx-auto">
      <h2
        v-motion
        :initial="{ opacity: 0, y: -10 }"
        :enter="{ opacity: 1, y: 0, transition: { duration: 800 } }"
        class="text-xs font-bold text-tea-green/60 tracking-[0.3em] uppercase mb-4"
      >
        拾梦茶舍 | 追忆之庭
      </h2>
      <h1
        v-motion
        :initial="{ opacity: 0, scale: 0.95 }"
        :enter="{ opacity: 1, scale: 1, transition: { duration: 1000, delay: 200 } }"
        class="font-serif text-4xl md:text-5xl lg:text-6xl text-tea-green relative inline-block leading-tight tracking-widest"
      >
        壹周年拾忆纪念
        <div class="absolute -bottom-6 left-1/2 -translate-x-1/2 w-32 h-px bg-linear-to-r from-transparent via-tea-green/30 to-transparent" />
      </h1>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
      <MemberCard
        v-for="(member, index) in members"
        :key="member.id"
        :member="member"
        :delay="0.1 * (index % 4)"
      />
    </div>
  </div>
</template>
