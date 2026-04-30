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

/** Fisher-Yates: 随机打乱数组元素 */
function shuffle<T>(arr: T[]): T[] {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

/** 随机返回 1 或 -1，用于左右方向随机 */
function randomSign(): number {
  return Math.random() > 0.5 ? 1 : -1;
}

// #region 位置生成
// 为每张浮动卡片找一个不与其他卡片重叠、
// 且不遮挡中心按钮的位置。三层降级策略：
//   1. 正常随机（严格间距）
//   2. 放宽间距（200 次后自动触发）
//   3. 兜底1：沿中心避让区边缘散布
//   4. 兜底2：硬放

/** 位置生成参数 */
const POSITION_CONFIG = {
  maxAttempts: 400,       // 正常随机的最大尝试次数
  relaxThreshold: 200,    // 第 N 次后开始放宽间距
  distX: 300,             // 水平最小间距（卡片宽 288 + 余量）
  distY: 100,             // 垂直最小间距（卡片高 60 + 余量）
  distXRelaxed: 250,      // 放宽后的水平间距
  distYRelaxed: 75,       // 放宽后的垂直间距
  safeZoneX: 280,         // 中心水平避让半径（按钮 + 缓冲）
  safeZoneY: 210,         // 中心垂直避让半径（按钮 + 标题 + 缓冲）
  fallbackSpread: 120,    // 兜底时从避让区边缘向外散布的范围
  fallbackAttempts: 30,   // 兜底阶段的最大尝试次数
};

/** 碰撞检测：椭圆形判定横长条形状（约 288×60px）*/
function hasCollision(
  x: number, y: number,
  existing: { x: number; y: number }[],
  distX: number,
  distY: number,
  safeZoneX: number,
  safeZoneY: number
): boolean {
  // 落在中心避让区内
  if (Math.abs(x) < safeZoneX && Math.abs(y) < safeZoneY) return true;
  // 与已有卡片的椭圆距离过近
  return existing.some(pos =>
    Math.abs(x - pos.x) < distX && Math.abs(y - pos.y) < distY
  );
}

/**
 * 为新卡片生成一个不重叠的位置
 * @param existing 已放置卡片的位置列表
 * @returns { x, y } 相对于屏幕中心的偏移量
 */
function generatePosition(existing: { x: number; y: number }[]) {
  const screenW = typeof window !== 'undefined' ? window.innerWidth : 1200;
  const screenH = typeof window !== 'undefined' ? window.innerHeight : 800;

  // 随机范围: 屏幕中心到边缘，减去卡片自身半宽和安全边距
  const maxX = (screenW / 2) - 144 - 30;
  const maxY = (screenH / 2) - 36 - 100;

  const {
    maxAttempts, relaxThreshold,
    distX, distY, distXRelaxed, distYRelaxed,
    safeZoneX, safeZoneY,
    fallbackSpread, fallbackAttempts
  } = POSITION_CONFIG;

  // 1. 正常随机，前 200 次严格间距，后放宽
  for (let i = 0; i < maxAttempts; i++) {
    const x = (Math.random() - 0.5) * maxX * 2;
    const y = (Math.random() - 0.5) * maxY * 2;
    const dx = i < relaxThreshold ? distX : distXRelaxed;
    const dy = i < relaxThreshold ? distY : distYRelaxed;
    if (!hasCollision(x, y, existing, dx, dy, safeZoneX, safeZoneY)) {
      return { x, y };
    }
  }

  // 2. 沿中心避让区边缘散布 + 一定随机偏移
  for (let i = 0; i < fallbackAttempts; i++) {
    const x = randomSign() * (safeZoneX + Math.random() * fallbackSpread);
    const y = (Math.random() - 0.5) * maxY * 2;
    if (!hasCollision(x, y, existing, distXRelaxed, distYRelaxed, safeZoneX, safeZoneY)) {
      return { x, y };
    }
  }

  // 3. 硬放
  return {
    x: randomSign() * (safeZoneX + 10),
    y: (Math.random() - 0.5) * maxY * 2
  };
}
// #endregion

// #region 生成卡片
// 随机打乱成员列表，取前 10 个作为初始显示
const initialMembers = shuffle(props.members).slice(0, 10);

// 逐个生成不重叠的位置
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

// #endregion

// #region 定时轮换
// 每 4 秒随机替换一张卡片，悬停时暂停

let interval: any;
const isHovered = ref(false);

onMounted(() => {
  interval = setInterval(() => {
    // 鼠标悬停在卡片上时暂停轮换
    if (isHovered.value) return;
    activeMembers.value = (() => {
      // 找出还没显示的成员
      const remaining = props.members.filter(
        (m: Member) => !activeMembers.value.some((am: any) => am.id === m.id)
      );
      if (remaining.length === 0) return activeMembers.value;

      // 随机选一个新成员
      const newMember = remaining[Math.floor(Math.random() * remaining.length)];

      // 随机替换一张现有卡片
      const next = [...activeMembers.value];
      const replaceIndex = Math.floor(Math.random() * next.length);

      // 新位置避开其他卡片 (不包含被替换的那张)
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
// #endregion
</script>

<template>
  <div class="relative min-h-[85vh] w-full flex items-center justify-center overflow-hidden">
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