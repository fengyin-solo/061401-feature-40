<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from 'vue'
import StatusPanel from '@/components/StatusPanel.vue'
import ActionButtons from '@/components/ActionButtons.vue'
import EventLog from '@/components/EventLog.vue'
import GameOverModal from '@/components/GameOverModal.vue'
import { useGame } from '@/composables/useGame'

const { state, highScore, canPerformAction, gatherWood, gatherStone, hunt, drink, restart } = useGame()

const isNewRecord = computed(() => state.value.turn >= highScore.value && state.value.turn > 0)

const MOBILE_BREAKPOINT = 768
const isCompact = ref(false)
const isManualCompact = ref<boolean | null>(null)

const statusCollapsed = ref(false)
const actionsCollapsed = ref(false)
const logsCollapsed = ref(false)

function checkScreenSize() {
  if (isManualCompact.value !== null) return
  isCompact.value = window.innerWidth < MOBILE_BREAKPOINT
}

function toggleCompact() {
  isManualCompact.value = !isCompact.value
  isCompact.value = isManualCompact.value
}

onMounted(() => {
  checkScreenSize()
  window.addEventListener('resize', checkScreenSize)
})

onUnmounted(() => {
  window.removeEventListener('resize', checkScreenSize)
})
</script>

<template>
  <div class="min-h-screen bg-game-bg text-white">
    <div class="absolute inset-0 overflow-hidden pointer-events-none">
      <div class="absolute top-0 left-1/4 w-96 h-96 bg-green-900/10 rounded-full blur-3xl"></div>
      <div class="absolute bottom-0 right-1/4 w-96 h-96 bg-blue-900/10 rounded-full blur-3xl"></div>
    </div>

    <div class="relative z-10 max-w-6xl mx-auto" :class="isCompact ? 'px-2 py-3' : 'px-4 py-8'">
      <header :class="isCompact ? 'text-center mb-3' : 'text-center mb-8'">
        <h1 :class="[
          'font-bold bg-gradient-to-r from-green-400 via-emerald-400 to-teal-400 bg-clip-text text-transparent',
          isCompact ? 'text-xl mb-1' : 'text-4xl mb-2'
        ]">
          🏕️ 荒野生存
        </h1>
        <p v-if="!isCompact" class="text-gray-400">在恶劣的荒野中尽可能生存更久</p>
      </header>

      <div :class="['flex justify-center', isCompact ? 'gap-2 mb-3' : 'gap-8 mb-8']">
        <div :class="[
          'bg-game-card/80 backdrop-blur rounded-xl border border-game-border',
          isCompact ? 'px-3 py-1.5' : 'px-6 py-3'
        ]">
          <span :class="['text-gray-400', isCompact ? 'text-xs' : 'text-sm']">当前回合</span>
          <p :class="['font-bold text-white tabular-nums', isCompact ? 'text-lg' : 'text-2xl']">{{ state.turn }}</p>
        </div>
        <div :class="[
          'bg-game-card/80 backdrop-blur rounded-xl border border-game-border',
          isCompact ? 'px-3 py-1.5' : 'px-6 py-3'
        ]">
          <span :class="['text-gray-400', isCompact ? 'text-xs' : 'text-sm']">最高纪录</span>
          <p :class="['font-bold text-yellow-400 tabular-nums', isCompact ? 'text-lg' : 'text-2xl']">🏆 {{ highScore }}</p>
        </div>
      </div>

      <div class="flex justify-center mb-3">
        <button
          @click="toggleCompact"
          class="px-3 py-1.5 text-xs rounded-lg bg-game-card border border-game-border text-gray-300 hover:bg-game-card-hover transition-colors"
        >
          {{ isCompact ? '🔍 正常模式' : '📱 紧凑模式' }}
        </button>
      </div>

      <div :class="['grid gap-4', isCompact ? 'grid-cols-1 gap-3' : 'grid-cols-1 lg:grid-cols-3 gap-6']">
        <div>
          <StatusPanel
            :health="state.health"
            :hunger="state.hunger"
            :thirst="state.thirst"
            :wood="state.wood"
            :stone="state.stone"
            :is-compact="isCompact"
            :collapsed="statusCollapsed"
            @toggle-collapse="statusCollapsed = !statusCollapsed"
          />
        </div>

        <div>
          <ActionButtons
            :can-gather-wood="canPerformAction('gatherWood')"
            :can-gather-stone="canPerformAction('gatherStone')"
            :can-hunt="canPerformAction('hunt')"
            :can-drink="canPerformAction('drink')"
            :disabled="state.isGameOver"
            :is-compact="isCompact"
            :collapsed="actionsCollapsed"
            @toggle-collapse="actionsCollapsed = !actionsCollapsed"
            @gather-wood="gatherWood"
            @gather-stone="gatherStone"
            @hunt="hunt"
            @drink="drink"
          />
        </div>

        <div>
          <EventLog
            :logs="state.logs"
            :is-compact="isCompact"
            :collapsed="logsCollapsed"
            @toggle-collapse="logsCollapsed = !logsCollapsed"
          />
        </div>
      </div>

      <footer :class="['text-center text-gray-500 text-sm', isCompact ? 'mt-3 text-xs' : 'mt-8']">
        <p>💡 提示：生命值归零或饥饿/口渴值满格则游戏结束</p>
      </footer>
    </div>

    <GameOverModal
      :show="state.isGameOver"
      :final-turn="state.turn"
      :high-score="highScore"
      :is-new-record="isNewRecord"
      @restart="restart"
    />
  </div>
</template>
