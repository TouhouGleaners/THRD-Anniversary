<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { Grid, Sparkles, X } from 'lucide-vue-next';
import AnniversaryGrid from './components/AnniversaryGrid.vue';
import OrbitView from './components/OrbitView.vue';
import MemberCard from './components/MemberCard.vue';
import type { Member } from './types';

type ViewMode = 'orbit' | 'grid';

const viewMode = ref<ViewMode>('orbit');
const selectedMember = ref<Member | null>(null);
const members = ref<Member[]>([]);
const isLoading = ref(true);

onMounted(async () => {
  try {
    const res = await fetch(`/1/members.json`);
    if (res.ok) {
      members.value = await res.json();
    } else {
      console.error('Failed to load members data');
    }
  } catch (e) {
    console.error(e);
  } finally {
    isLoading.value = false;
  }
});

const handleEnterDream = () => {
  if (members.value.length === 0) return;
  const randomMember = members.value[Math.floor(Math.random() * members.value.length)];
  selectedMember.value = randomMember;
};

const setMode = (mode: ViewMode) => {
  viewMode.value = mode;
  selectedMember.value = null;
};
</script>

<template>
  <div class="min-h-screen flex flex-col font-sans selection:bg-amber/20 selection:text-amber">
    <!-- Navigation Header -->
    <header class="fixed top-0 w-full z-50 glass border-b border-tea-green/5 transition-all duration-700">
      <div class="max-w-7xl mx-auto px-8 h-16 flex items-center justify-between">
        <div class="font-serif text-xl md:text-2xl text-tea-green tracking-tighter cursor-pointer hover:opacity-70 transition-opacity" @click="setMode('orbit')">
          THRD | 拾遗梦茶馆
        </div>

        <nav class="hidden md:flex items-center gap-10 absolute left-1/2 -translate-x-1/2">
          <button
            class="font-serif text-lg transition-all duration-500 hover:text-tea-green relative"
            :class="viewMode === 'orbit' ? 'text-tea-green' : 'text-charcoal/40'"
            @click="setMode('orbit')"
          >
            漫游
            <div v-if="viewMode === 'orbit'" class="absolute -bottom-1 left-0 right-0 h-[1.5px] bg-tea-green" />
          </button>
          <button
            class="font-serif text-lg transition-all duration-500 hover:text-tea-green relative"
            :class="viewMode === 'grid' ? 'text-tea-green' : 'text-charcoal/40'"
            @click="setMode('grid')"
          >
            群像
            <div v-if="viewMode === 'grid'" class="absolute -bottom-1 left-0 right-0 h-[1.5px] bg-tea-green" />
          </button>
        </nav>
      </div>
    </header>

    <!-- Main Content Area -->
    <main class="grow pt-16 flex flex-col items-center justify-center relative">
      <div v-if="isLoading" class="flex flex-col items-center justify-center text-tea-green animate-pulse w-full h-full min-h-[60vh]">
        <span class="font-serif text-xl tracking-[0.2em]">少女祈祷中...</span>
      </div>
      <Transition v-else name="fade" mode="out-in">
        <OrbitView v-if="viewMode === 'orbit'" :onEnter="handleEnterDream" :members="members" @select="selectedMember = $event" class="w-full" />
        <AnniversaryGrid v-else :members="members" class="w-full" />
      </Transition>

      <!-- Liquid Glass Modal for Selected Dreamer -->
      <Transition name="modal">
        <div
          v-if="selectedMember"
          class="fixed inset-0 z-100 flex items-center justify-center p-6 bg-parchment/60 backdrop-blur-xl"
          @click="selectedMember = null"
        >
          <div
            class="w-full max-w-lg relative"
            @click.stop
          >
            <div class="relative group">
              <MemberCard
                :member="selectedMember"
                class="flex-col! items-center! text-center p-10 md:p-14 gap-8! shadow-[0_40px_100px_rgba(6,27,14,0.3)] mx-auto border"
              />

              <button
                @click="selectedMember = null"
                class="absolute top-6 right-6 p-2 rounded-full hover:bg-tea-green/5 transition-all text-charcoal/40 hover:text-tea-green hover:rotate-90 duration-500 z-50 mix-blend-difference"
              >
                <X :size="24" />
              </button>

              <div class="absolute -bottom-16 left-1/2 -translate-x-1/2 text-[10px] uppercase font-bold text-center w-full leading-relaxed tracking-[0.4em] text-tea-green/40 opacity-0 group-hover:opacity-100 transition-opacity duration-700">
                浮生一瞥<br/>
                <span class="text-[8px]">点击任意处重返梦境</span>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </main>

    <!-- Footer -->
    <footer class="py-6 border-t border-tea-green/5 bg-white/30 backdrop-blur-md">
      <div class="max-w-7xl mx-auto px-8 flex justify-center items-center">
        <div class="text-xs text-charcoal/40 font-serif tracking-wide text-center leading-relaxed">
          © 2026 Miku_oso · THRD | All rights reserved.
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s, filter 1s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  filter: blur(10px);
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.5s;
}
.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}
.modal-enter-active > div {
  animation: spring-in 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}
.modal-leave-active > div {
  animation: spring-out 0.4s ease-in;
}

@keyframes spring-in {
  0% { transform: scale(0.9) translateY(20px); opacity: 0; }
  100% { transform: scale(1) translateY(0); opacity: 1; }
}
@keyframes spring-out {
  0% { transform: scale(1) translateY(0); opacity: 1; }
  100% { transform: scale(0.9) translateY(20px); opacity: 0; }
}
</style>
