<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import type { Member } from '../../types';
import OrbitingCard from './OrbitingCard.vue';

const props = defineProps<{
  onEnter: () => void;
  members: Member[];
}>();

const emit = defineEmits<{
  (e: 'select', member: Member): void
}>();

function generatePosition(existingPositions: {x: number, y: number}[]) {
  let valid = false;
  let x = 0;
  let y = 0;
  let attempts = 0;

  const screenW = typeof window !== 'undefined' ? window.innerWidth : 1200;
  const screenH = typeof window !== 'undefined' ? window.innerHeight : 800;

  const maxX = (screenW / 2) - 144 - 30;  // 30为安全边缘
  const maxY = (screenH / 2) - 36 - 100;  // 100为页眉页脚预留

  const safeZoneX = 280;  // 水平避让: 卡片半径(144) + 按钮半径(112) + 缓冲(20) = 276
  const safeZoneY = 210;  // 垂直避让: 卡片半径(36) + 按钮半径(112) + 标题高度缓冲(60) = 208

  while(!valid && attempts < 400) {
    x = (Math.random() - 0.5) * (maxX * 2);
    y = (Math.random() - 0.5) * (maxY * 2);

    // 检查中心矩形避让区
    let overlapsCenter = Math.abs(x) < safeZoneX && Math.abs(y) < safeZoneY;

    // 检查与其他卡片的碰撞
    let minCardDistX = 320;
    let minCardDistY = 100;

    // 动态调整间距: 如果多次尝试后仍未满足需求，允许卡片稍微靠拢
    if (attempts > 200) {
      minCardDistX = 260;
      minCardDistY = 80;
    }

    let overlapsCard = existingPositions.some(pos => {
      return Math.abs(x - pos.x) < minCardDistX && Math.abs(y - pos.y) < minCardDistY;
    });

    if (!overlapsCenter && !overlapsCard) {
       valid = true;
    }
    attempts++;
  }

  if (!valid) {
    x = (Math.random() > 0.5 ? 1 : -1) * (safeZoneX + 10);
    y = (Math.random() - 0.5) * (maxY * 2);
  }

  return { x, y };
}

// Initial cards
const initialMembers = props.members.slice(0, 8);
const placedMembers: any[] = [];
for (let i = 0; i < initialMembers.length; i++) {
  const pos = generatePosition(placedMembers);
  placedMembers.push({
    ...initialMembers[i],
    ...pos,
    keyId: `${initialMembers[i].id}-${Date.now()}-${i}`
  });
}

const activeMembers = ref(placedMembers);

let interval: any;
const isHovered = ref(false);

onMounted(() => {
  interval = setInterval(() => {
    if (isHovered.value) return;
    activeMembers.value = (() => {
      const remaining = props.members.filter((m: Member) => !activeMembers.value.some((am: any) => am.id === m.id));
      if (remaining.length === 0) return activeMembers.value;
      const newMember = remaining[Math.floor(Math.random() * remaining.length)];

      const next = [...activeMembers.value];
      const replaceIndex = Math.floor(Math.random() * next.length);

      // Get positions of all OTHER cards so the new one doesn't overlap them
      const otherPositions = next.filter((_, i) => i !== replaceIndex).map((m: any) => ({x: m.x, y: m.y}));
      const newPos = generatePosition(otherPositions);

      next[replaceIndex] = {
        ...newMember,
        ...newPos,
        keyId: `${newMember.id}-${Date.now()}`
      };
      return next;
    })();
  }, 4000);
});

onUnmounted(() => {
  clearInterval(interval);
});
</script>

<template>
  <div class="relative min-h-[85vh] w-full flex items-center justify-center overflow-hidden">
    <!-- Dynamic Background Blurs -->
    <div class="absolute top-1/4 left-1/4 w-96 h-96 bg-amber/5 rounded-full blur-[120px] -z-10 animate-pulse" />
    <div class="absolute bottom-1/4 right-1/4 w-96 h-96 bg-tea-green/5 rounded-full blur-[120px] -z-10 animate-pulse" style="animation-delay: 2s;" />

    <!-- Central Interactive Core -->
    <div class="relative z-60 flex flex-col items-center justify-center text-center animate-fade-in-up">
      <h2 class="font-serif italic text-xl md:text-2xl text-tea-green/70 mb-4 tracking-[0.15em]">
        共筑之梦 · 壹周年
      </h2>
      <button
        @click="onEnter"
        class="cursor-pointer group relative w-56 h-56 rounded-full bg-tea-green text-parchment flex flex-col items-center justify-center shadow-2xl transition-all duration-700 hover:shadow-amber/20 hover:scale-105 active:scale-95 overflow-hidden"
      >
        <div class="absolute inset-1 rounded-full border border-parchment/10 group-hover:border-parchment/30 transition-colors pointer-events-none" />
        <span class="font-serif text-5xl mb-2 tracking-tighter">入梦</span>
        <span class="text-[10px] uppercase font-bold tracking-[0.2em] text-parchment/60 group-hover:text-amber transition-colors">
          坠入梦境
        </span>
        <div class="absolute inset-0 bg-parchment/5 pointer-events-none opacity-0 group-hover:opacity-100 transition-opacity" />
      </button>
      <p class="mt-8 text-charcoal/60 font-serif italic text-sm max-w-xs leading-relaxed tracking-wider">
        "聆听那些构筑静谧之庭的回声"
      </p>
    </div>

    <!-- Randomly positioned floating cards -->
    <div class="absolute inset-0 pointer-events-none z-10">
      <TransitionGroup name="float-fade">
        <template v-for="(member, i) in activeMembers" :key="member.keyId">
          <OrbitingCard
            :member="member"
            :x="member.x"
            :y="member.y"
            :index="i"
            @mouseenter="isHovered = true"
            @mouseleave="isHovered = false"
            @click="emit('select', member)"
          />
        </template>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped>
.animate-fade-in-up {
  animation: fadeInUp 1.5s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(20px) scale(0.95); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

.float-fade-enter-active,
.float-fade-leave-active {
  transition: opacity 2s ease, filter 2s ease, transform 2s ease;
}
.float-fade-enter-from,
.float-fade-leave-to {
  opacity: 0;
  filter: blur(8px);
  transform: scale(0.9) translateY(10px);
}
</style>
