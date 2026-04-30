<script setup lang="ts">
import { computed } from 'vue';
import type { Member } from '../types';

const props = defineProps<{
  member: Member;
  className?: string;
  delay?: number;
}>();

const companionDays = computed(() => {
  if (!props.member.joinedAt) return null;
  const joined = new Date(props.member.joinedAt).getTime();
  const now = Date.now();
  return Math.floor((now - joined) / 86400000)
});
</script>

<template>
  <div
    v-motion
    :initial="{ opacity: 0, y: 30 }"
    :enter="{ opacity: 1, y: 0, transition: { duration: 1000, delay: (delay || 0) * 100 } }"
    class="relative flex flex-row gap-5 p-6 md:p-8 items-start transition-all duration-700 ease-out hover:-translate-y-3 hover:scale-[1.03] group liquid-card"
    :class="className"
  >
    <!-- Liquid internal blobs and glossy edge -->
    <div class="absolute inset-0 overflow-hidden pointer-events-none" style="border-radius: inherit;">
      <div class="absolute -top-12 -left-12 w-48 h-48 bg-linear-to-br from-amber/20 opacity-80 to-transparent rounded-full blur-[25px] animate-flow-1 z-0" />
      <div class="absolute -bottom-16 -right-12 w-64 h-64 bg-linear-to-tl from-tea-green/20 opacity-80 to-transparent rounded-full blur-[30px] animate-flow-2 z-0" />
      <div class="absolute top-0 left-0 right-0 h-[40%] bg-linear-to-b from-white/60 to-transparent z-0 opacity-80 mix-blend-overlay" />
    </div>

    <!-- Avatar Image -->
    <div class="relative z-10 w-20 h-20 shrink-0 border-[1.5px] border-white/60 shadow-[inset_0_2px_5px_rgba(255,255,255,0.5),0_8px_20px_rgba(0,0,0,0.08)] group-hover:border-amber/50 group-hover:shadow-[0_8px_25px_rgba(130,85,22,0.15)] transition-all duration-700 overflow-hidden liquid-image">
      <img
        :src="member.avatar"
        :alt="member.name"
        :style="{ objectPosition: member.imagePosition || 'center' }"
        class="w-full h-full object-cover grayscale-15 group-hover:grayscale-0 transition-all duration-700 scale-105 group-hover:scale-110"
        referrerpolicy="no-referrer"
      />
    </div>

    <!-- Text Content -->
    <div class="relative z-10 flex flex-col gap-3 grow">
      <div class="flex flex-col items-start gap-1">
        <span
          v-if="member.title"
          class="text-[9px] uppercase font-bold tracking-[0.25em] px-3 py-1 rounded-full border shadow-sm transition-colors duration-500 bg-white/50 text-tea-green border-tea-green/20"
        >
          {{ member.title }}
        </span>
        <h3 class="font-serif text-2xl md:text-3xl tracking-tight text-tea-green group-hover:text-amber transition-colors duration-500 drop-shadow-sm">
          {{ member.name }}
        </h3>
      </div>
      <p class="text-[14px] md:text-[15px] text-charcoal/85 leading-relaxed font-serif italic tracking-wide">
        "{{ member.quote }}"
      </p>
      <span
        v-if="companionDays !== null"
        class="text-[11px] text-charcoal/30 tracking-wider font-sans mt-0.5"
      >
        陪伴了 {{ companionDays }} 天
      </span>
    </div>
  </div>
</template>

<style scoped>
.liquid-card {
  border-radius: 40px 25px 45px 20px;
  box-shadow:
    inset 0 4px 15px rgba(255,255,255,0.8),
    inset 0 -4px 15px rgba(6,27,14,0.03),
    0 15px 35px rgba(6,27,14,0.06);
  border: 1px solid rgba(255,255,255,0.6);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  background: linear-gradient(135deg, rgba(255,255,255,0.5) 0%, rgba(255,255,255,0.15) 100%);
}

.liquid-image {
  border-radius: 25px 15px 30px 15px;
}

@keyframes flow-1 {
  0% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(15px, 10px) scale(1.1); }
  66% { transform: translate(-10px, 15px) scale(0.9); }
  100% { transform: translate(0, 0) scale(1); }
}

@keyframes flow-2 {
  0% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(-20px, -15px) scale(1.05); }
  66% { transform: translate(10px, -20px) scale(1.15); }
  100% { transform: translate(0, 0) scale(1); }
}

.animate-flow-1 {
  animation: flow-1 8s ease-in-out infinite;
}
.animate-flow-2 {
  animation: flow-2 10s ease-in-out infinite reverse;
}
</style>
