<script setup lang="ts">
import type { Member } from '../types';

defineProps<{
  members: Member[];
}>();

const emit = defineEmits<{
  (e: 'select', member: Member): void;
}>();
</script>

<template>
  <div class="max-w-4xl mx-auto w-full px-6 py-12 md:py-20 lg:py-24">
    <header class="flex flex-col items-center mb-20 text-center w-full mx-auto">
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

    <!-- 展墙列表 -->
    <div class="flex flex-col">
      <div
        v-for="(member, index) in members"
        :key="member.id"
        v-motion
        :initial="{ opacity: 0, y: 16 }"
        :enter="{ opacity: 1, y: 0, transition: { duration: 500, delay: index * 60 } }"
        @click="emit('select', member)"
        class="group flex items-start gap-5 py-6 md:py-7 border-b border-tea-green/10 last:border-b-0 cursor-pointer transition-colors duration-300 hover:bg-white/40 rounded-xl px-4 -mx-4"
      >
        <!-- 头像 -->
        <div class="w-11 h-11 rounded-full overflow-hidden shrink-0 border border-white/50 shadow-sm grayscale-20 group-hover:grayscale-0 transition-all duration-500">
          <img
            :src="member.avatar"
            :alt="member.name"
            :style="{ objectPosition: member.imagePosition || 'center' }"
            class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-500"
            referrerpolicy="no-referrer"
          />
        </div>

        <!-- 内容 -->
        <div class="flex flex-col md:flex-row md:items-center gap-1 md:gap-6 grow min-w-0">
          <!-- 称号 + 昵称 -->
          <div class="flex flex-col gap-0.5 shrink-0 md:w-44">
            <span
              v-if="member.title"
              class="text-[9px] uppercase font-bold tracking-[0.25em] text-charcoal/35"
            >
              {{ member.title }}
            </span>
            <h3 class="font-serif text-base md:text-lg text-tea-green group-hover:text-amber transition-colors duration-300 tracking-tight">
              {{ member.name }}
            </h3>
          </div>

          <!-- 引言 -->
          <p class="font-serif italic text-[13px] text-charcoal/50 leading-relaxed tracking-wide group-hover:text-charcoal/70 transition-colors duration-300 grow">
            "{{ member.quote }}"
          </p>
        </div>
      </div>
    </div>
  </div>
</template>