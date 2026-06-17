<script setup lang="ts">
import { computed } from 'vue'

interface StatItem {
  label: string
  value: number
  max: number
  icon: string
  color: string
  barColor: string
  isReverse?: boolean
}

interface Props {
  health: number
  hunger: number
  thirst: number
  wood: number
  stone: number
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

const stats = computed<StatItem[]>(() => [
  {
    label: '生命值',
    value: props.health,
    max: 100,
    icon: '❤️',
    color: 'text-red-400',
    barColor: 'bg-red-500',
  },
  {
    label: '饥饿值',
    value: props.hunger,
    max: 100,
    icon: '🍖',
    color: 'text-orange-400',
    barColor: 'bg-orange-500',
    isReverse: true,
  },
  {
    label: '口渴值',
    value: props.thirst,
    max: 100,
    icon: '💧',
    color: 'text-blue-400',
    barColor: 'bg-blue-500',
    isReverse: true,
  },
  {
    label: '木材',
    value: props.wood,
    max: 100,
    icon: '🪵',
    color: 'text-amber-600',
    barColor: 'bg-amber-600',
  },
  {
    label: '石头',
    value: props.stone,
    max: 100,
    icon: '🪨',
    color: 'text-gray-400',
    barColor: 'bg-gray-400',
  },
])

const summaryText = computed(() => {
  const critical = stats.value.filter(s => isDanger(s.value, s.max, s.isReverse))
  if (critical.length > 0) {
    return critical.map(s => `${s.icon}${Math.round(s.value)}`).join(' ')
  }
  return stats.value.slice(0, 3).map(s => `${s.icon}${Math.round(s.value)}`).join(' ')
})

function getBarWidth(value: number, max: number): string {
  const percent = Math.max(0, Math.min(100, (value / max) * 100))
  return `${percent}%`
}

function isDanger(value: number, max: number, isReverse?: boolean): boolean {
  const percent = (value / max) * 100
  if (isReverse) {
    return percent >= 80
  }
  return percent <= 20
}
</script>

<template>
  <div class="bg-game-card rounded-2xl border border-game-border shadow-xl overflow-hidden">
    <button
      @click="emit('toggle-collapse')"
      class="w-full flex items-center justify-between text-left transition-colors hover:bg-game-card-hover"
      :class="isCompact ? 'px-3 py-2' : 'px-6 py-4'"
    >
      <h2 :class="['font-bold text-white flex items-center gap-2', isCompact ? 'text-base' : 'text-xl']">
        <span>📊</span>
        <span>生存状态</span>
      </h2>
      <div class="flex items-center gap-2">
        <span v-if="collapsed" class="text-xs text-gray-400">{{ summaryText }}</span>
        <span :class="['text-gray-400 transition-transform duration-200', collapsed ? '' : 'rotate-180']">▼</span>
      </div>
    </button>
    <div
      :class="[
        'transition-all duration-300 ease-in-out overflow-hidden',
        collapsed ? 'max-h-0' : isCompact ? 'max-h-96' : 'max-h-[500px]'
      ]"
    >
      <div :class="['space-y-4', isCompact ? 'px-3 pb-3 space-y-2' : 'px-6 pb-6']">
        <div
          v-for="stat in stats"
          :key="stat.label"
          class="group"
        >
          <div class="flex items-center justify-between" :class="isCompact ? 'mb-0.5' : 'mb-1.5'">
            <div class="flex items-center gap-2">
              <span :class="isCompact ? 'text-sm' : 'text-lg'">{{ stat.icon }}</span>
              <span :class="[stat.color, 'font-medium', isCompact ? 'text-xs' : 'text-sm']">{{ stat.label }}</span>
            </div>
            <span
              :class="[
                stat.color,
                'font-bold tabular-nums',
                isCompact ? 'text-xs' : 'text-sm',
                isDanger(stat.value, stat.max, stat.isReverse) ? 'animate-pulse-soft' : '',
              ]"
            >
              {{ Math.round(stat.value) }}
            </span>
          </div>
          <div
            :class="[
              'bg-gray-800 rounded-full overflow-hidden',
              isCompact ? 'h-1.5' : 'h-2.5'
            ]"
          >
            <div
              :class="[stat.barColor, 'h-full rounded-full transition-all duration-300 ease-out']"
              :style="{ width: getBarWidth(stat.value, stat.max) }"
            ></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
