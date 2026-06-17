<script setup lang="ts">
import { ref, watch, nextTick, computed } from 'vue'
import type { LogEntry } from '@/types/game'

interface Props {
  logs: LogEntry[]
  isCompact?: boolean
  collapsed?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  isCompact: false,
  collapsed: false,
})

const emit = defineEmits<{
  'toggle-collapse': []
}>()

const logContainer = ref<HTMLElement | null>(null)

function getLogColor(type: LogEntry['type']): string {
  switch (type) {
    case 'good':
      return 'text-green-400'
    case 'bad':
      return 'text-red-400'
    case 'system':
      return 'text-purple-400'
    case 'event':
      return 'text-yellow-400'
    default:
      return 'text-gray-300'
  }
}

function getLogIcon(type: LogEntry['type']): string {
  switch (type) {
    case 'good':
      return '✨'
    case 'bad':
      return '⚠️'
    case 'system':
      return '📢'
    case 'event':
      return '🎲'
    default:
      return '▶️'
  }
}

const latestLog = computed(() => {
  if (props.logs.length === 0) return null
  return props.logs[0]
})

const logCount = computed(() => props.logs.length)

watch(
  () => props.logs.length,
  async () => {
    await nextTick()
    if (logContainer.value) {
      logContainer.value.scrollTop = 0
    }
  }
)
</script>

<template>
  <div class="bg-game-card rounded-2xl border border-game-border shadow-xl overflow-hidden">
    <button
      @click="emit('toggle-collapse')"
      class="w-full flex items-center justify-between text-left transition-colors hover:bg-game-card-hover"
      :class="isCompact ? 'px-3 py-2' : 'px-6 py-4'"
    >
      <h2 :class="['font-bold text-white flex items-center gap-2', isCompact ? 'text-base' : 'text-xl']">
        <span>📜</span>
        <span>事件日志</span>
        <span class="text-xs text-gray-400 font-normal">({{ logCount }})</span>
      </h2>
      <div class="flex items-center gap-2 max-w-[60%]">
        <span
          v-if="collapsed && latestLog"
          :class="[getLogColor(latestLog.type), 'text-xs truncate']"
        >
          {{ getLogIcon(latestLog.type) }} {{ latestLog.text }}
        </span>
        <span :class="['text-gray-400 transition-transform duration-200 flex-shrink-0', collapsed ? '' : 'rotate-180']">▼</span>
      </div>
    </button>
    <div
      :class="[
        'transition-all duration-300 ease-in-out overflow-hidden',
        collapsed ? 'max-h-0' : isCompact ? 'max-h-48' : 'max-h-[500px]'
      ]"
    >
      <div :class="['flex flex-col', isCompact ? 'px-2 pb-2' : 'px-6 pb-6']">
        <div
          ref="logContainer"
          class="overflow-y-auto scrollbar-thin scrollbar-thumb-gray-700 scrollbar-track-transparent"
          :class="isCompact ? 'space-y-1 pr-1' : 'space-y-2 pr-2'"
          :style="{ maxHeight: isCompact ? '180px' : '400px' }"
        >
          <div
            v-for="log in logs"
            :key="log.id"
            :class="[
              getLogColor(log.type),
              'animate-slide-up flex items-start gap-2',
              isCompact ? 'text-xs leading-snug' : 'text-sm leading-relaxed'
            ]"
          >
            <span class="flex-shrink-0">{{ getLogIcon(log.type) }}</span>
            <span>{{ log.text }}</span>
          </div>
          <div v-if="logs.length === 0" :class="['text-gray-500 text-center', isCompact ? 'text-xs py-4' : 'text-sm py-8']">
            暂无日志...
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.scrollbar-thin::-webkit-scrollbar {
  width: 6px;
}

.scrollbar-thin::-webkit-scrollbar-track {
  background: transparent;
}

.scrollbar-thin::-webkit-scrollbar-thumb {
  background-color: #374151;
  border-radius: 3px;
}

.scrollbar-thin::-webkit-scrollbar-thumb:hover {
  background-color: #4b5563;
}
</style>
