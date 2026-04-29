<script setup lang="ts">
const props = defineProps<{
  member: any;
  x: number;
  y: number;
  index: number;
}>();

const emit = defineEmits<{
  (e: 'mouseenter'): void;
  (e: 'mouseleave'): void;
  (e: 'click'): void;
}>();
</script>

<template>
  <div
    class="absolute pointer-events-auto cursor-pointer group"
    @mouseenter="emit('mouseenter')"
    @mouseleave="emit('mouseleave')"
    @click="emit('click')"
    :style="{
      left: `calc(50% + ${x}px)`,
      top: `calc(50% + ${y}px)`,
      marginLeft: '-144px',
      marginTop: '-36px'
    }"
  >
    <div class="flex items-center w-72 glass rounded-full p-2 pr-6 border border-white/40 shadow-ambient transition-all duration-500 hover:bg-white/80 animate-float hover:scale-105 hover:-translate-y-2 hover:shadow-2xl z-50 group-hover:z-100" :style="{ animationDelay: `${index * 1.5}s` }">
      <div class="w-14 h-14 rounded-full overflow-hidden shrink-0 border-2 border-white shadow-inner">
        <img :src="member.avatar" :alt="member.name" :style="{ objectPosition: member.imagePosition || 'center' }" class="w-full h-full object-cover" referrerpolicy="no-referrer" />
      </div>
      <div class="ml-4 grow truncate px-1">
        <div class="flex flex-col">
          <span class="font-serif text-sm text-tea-green font-medium truncate">{{ member.name }}</span>
          <span class="text-[9px] uppercase tracking-widest text-amber font-bold">{{ member.role }}</span>
        </div>
        <p class="text-[11px] text-charcoal/70 italic truncate mt-0.5">"{{ member.quote }}"</p>
      </div>
    </div>
  </div>
</template>
