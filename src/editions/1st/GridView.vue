<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import type { Member } from '../../types';

defineProps<{
  members: Member[];
}>();

const emit = defineEmits<{
  (e: 'select', member: Member): void;
}>();

// 列数切换（仅桌面端）
const columns = ref<1 | 2>(1);

// ── 时间轴 ──

interface TimelineEntry {
  date: string;
  title: string;
  type: string;
}

const timeline = ref<TimelineEntry[]>([]);

onMounted(async () => {
  try {
    const res = await fetch('editions/1st/timeline.json');
    if (res.ok) {
      timeline.value = await res.json();
    }
  } catch (e) {
    console.error(e);
  }
});

/** 按日期升序 */
const sortedTimeline = computed(() => {
  return [...timeline.value].sort((a, b) => {
    const parse = (s: string) => {
      const [y, m, d] = s.split('.').map(Number);
      return new Date(y, m - 1, d).getTime();
    };
    return parse(a.date) - parse(b.date);
  });
});

/** 按年份分组 */
const groupedTimeline = computed(() => {
  const groups: { year: number; entries: TimelineEntry[] }[] = [];
  for (const entry of sortedTimeline.value) {
    const year = parseInt(entry.date.split('.')[0]);
    const group = groups.find(g => g.year === year);
    if (group) {
      group.entries.push(entry);
    } else {
      groups.push({ year, entries: [entry] });
    }
  }
  return groups;
});
</script>

<template>
  <div :class="['mx-auto w-full px-6 py-12 md:py-20 lg:py-24 transition-all duration-500', columns === 1 ? 'max-w-4xl' : 'max-w-6xl']">
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
     <!-- 列数切换 -->
      <div class="hidden md:flex items-center justify-end gap-1 mb-8">
        <button
          @click="columns = 1"
          class="text-xs font-serif tracking-wider transition-all duration-300 px-3 py-1 rounded-full cursor-pointer"
          :class="columns === 1 ? 'text-parchment bg-tea-green' : 'text-charcoal/30 hover:text-tea-green'"
        >
          单列
        </button>
        <button
          @click="columns = 2"
          class="text-xs font-serif tracking-wider transition-all duration-300 px-3 py-1 rounded-full cursor-pointer"
          :class="columns === 2 ? 'text-parchment bg-tea-green' : 'text-charcoal/30 hover:text-tea-green'"
        >
          双列
        </button>
      </div>
    <div :class="columns === 1 ? 'flex flex-col' : 'grid grid-cols-2 gap-x-6 gap-y-1'">
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
        <div :class="['flex gap-1 grow min-w-0', columns === 1 ? 'flex-col md:flex-row md:items-center md:gap-6' : 'flex-col gap-0.5']">
          <!-- 称号 + 昵称 -->
          <div :class="['flex flex-col gap-0.5 shrink-0', columns === 1 ? 'md:w-44' : '']">
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
          <p :class="['font-serif italic text-[13px] text-charcoal/50 leading-relaxed tracking-wide group-hover:text-charcoal/70 transition-colors duration-300 grow', columns === 2 ? 'line-clamp-2' : '']">
            "{{ member.quote }}"
          </p>
        </div>
      </div>
    </div>
    <!-- ═══ 分隔 ═══ -->
    <div class="flex items-center gap-4 my-16 md:my-24">
      <div class="grow h-px bg-linear-to-r from-transparent via-tea-green/10 to-transparent" />
      <span class="text-charcoal/20 text-sm">✦</span>
      <div class="grow h-px bg-linear-to-r from-transparent via-tea-green/10 to-transparent" />
    </div>

    <!-- ═══ 时间轴 ═══ -->
    <section>
      <div class="flex flex-col items-center mb-16 text-center">
        <h2
          v-motion
          :initial="{ opacity: 0, y: -10 }"
          :enter="{ opacity: 1, y: 0, transition: { duration: 800 } }"
          class="text-xs font-bold text-tea-green/60 tracking-[0.3em] uppercase mb-4"
        >
          拾遗编年
        </h2>
        <h3
          v-motion
          :initial="{ opacity: 0, scale: 0.95 }"
          :enter="{ opacity: 1, scale: 1, transition: { duration: 1000, delay: 200 } }"
          class="font-serif text-2xl md:text-3xl text-tea-green tracking-widest"
        >
          这一年，我们走过的路
        </h3>
      </div>

      <div class="relative pl-8 md:pl-12">
        <!-- 竖线 -->
        <div class="absolute left-3 md:left-5 top-0 bottom-0 w-px bg-tea-green/10" />

        <div v-for="group in groupedTimeline" :key="group.year" class="mb-12 last:mb-0">
          <!-- 年份标题 -->
          <div class="relative mb-6 flex items-center gap-3">
            <div class="absolute -left-8 md:-left-12 w-3 h-3 md:w-3.5 md:h-3.5 rounded-full bg-amber/30 border-[1.5px] border-amber/50" />
            <h4 class="font-serif text-xl md:text-2xl text-amber tracking-wider">
              {{ group.year }}
            </h4>
            <span class="text-[10px] text-charcoal/20 tracking-wider font-sans">
              {{ group.entries.length }} 项
            </span>
          </div>

          <!-- 条目 -->
          <div
            v-for="(entry, idx) in group.entries"
            :key="entry.date + entry.title"
            v-motion
            :initial="{ opacity: 0, x: -16 }"
            :enter="{ opacity: 1, x: 0, transition: { duration: 400, delay: idx * 60 } }"
            class="relative flex items-start gap-4 md:gap-6 mb-4 group"
          >
            <!-- 节点 -->
            <div class="absolute -left-8 md:-left-12 top-2 w-2 h-2 rounded-full bg-tea-green/15 group-hover:bg-amber/60 transition-colors duration-300" />

            <!-- 日期 -->
            <span class="text-xs text-charcoal/35 font-sans tracking-wider shrink-0 w-20 md:w-24 pt-0.5">
              {{ entry.date }}
            </span>

            <!-- 内容 -->
            <div class="flex flex-col gap-0.5">
              <span class="font-serif text-sm md:text-base text-tea-green/80 group-hover:text-tea-green transition-colors duration-300 leading-snug">
                {{ entry.title }}
              </span>
              <span class="text-[10px] uppercase tracking-[0.2em] text-charcoal/25 font-bold">
                {{ entry.type }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>