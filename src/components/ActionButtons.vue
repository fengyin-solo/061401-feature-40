<script setup lang="ts">
import { computed } from 'vue'

interface ActionButton {
  label: string
  icon: string
  description: string
  action: () => void
  disabled: boolean
  bgClass: string
  hoverClass: string
}

interface Props {
  canGatherWood: boolean
  canGatherStone: boolean
  canHunt: boolean
  canDrink: boolean
  disabled: boolean
  isCompact?: boolean
  collapsed?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  isCompact: false,
  collapsed: false,
})

const emit = defineEmits<{
  gatherWood: []
  gatherStone: []
  hunt: []
  drink: []
  'toggle-collapse': []
}>()

const buttons: ActionButton[] = [
  {
    label: '采集木头',
    icon: '🪵',
    description: '获得木材，消耗体力',
    action: () => emit('gatherWood'),
    disabled: false,
    bgClass: 'bg-amber-900/40',
    hoverClass: 'hover:bg-amber-800/60',
  },
  {
    label: '采集石头',
    icon: '🪨',
    description: '获得石头，消耗体力',
    action: () => emit('gatherStone'),
    disabled: false,
    bgClass: 'bg-gray-700/40',
    hoverClass: 'hover:bg-gray-600/60',
  },
  {
    label: '打猎',
    icon: '🏹',
    description: '回复生命，增加饥饿，消耗木材',
    action: () => emit('hunt'),
    disabled: false,
    bgClass: 'bg-red-900/40',
    hoverClass: 'hover:bg-red-800/60',
  },
  {
    label: '喝水',
    icon: '💧',
    description: '减少口渴，消耗木材烧水',
    action: () => emit('drink'),
    disabled: false,
    bgClass: 'bg-blue-900/40',
    hoverClass: 'hover:bg-blue-800/60',
  },
]

function isButtonDisabled(index: number): boolean {
  if (props.disabled) return true
  const canActions = [props.canGatherWood, props.canGatherStone, props.canHunt, props.canDrink]
  return !canActions[index]
}

const availableCount = computed(() => {
  const canActions = [props.canGatherWood, props.canGatherStone, props.canHunt, props.canDrink]
  return canActions.filter(Boolean).length
})

const availableIcons = computed(() => {
  const result: string[] = []
  if (props.canGatherWood) result.push('🪵')
  if (props.canGatherStone) result.push('🪨')
  if (props.canHunt) result.push('🏹')
  if (props.canDrink) result.push('💧')
  return result.join(' ')
})
</script>

<template>
  <div class="bg-game-card rounded-2xl border border-game-border shadow-xl overflow-hidden">
    <button
      @click="emit('toggle-collapse')"
      class="w-full flex items-center justify-between text-left transition-colors hover:bg-game-card-hover"
      :class="isCompact ? 'px-3 py-2' : 'px-6 py-4'"
    >
      <h2 :class="['font-bold text-white flex items-center gap-2', isCompact ? 'text-base' : 'text-xl']">
        <span>⚡</span>
        <span>行动</span>
      </h2>
      <div class="flex items-center gap-2">
        <span v-if="collapsed" class="text-xs text-gray-400">
          {{ availableIcons }} ({{ availableCount }}/4)
        </span>
        <span :class="['text-gray-400 transition-transform duration-200', collapsed ? '' : 'rotate-180']">▼</span>
      </div>
    </button>
    <div
      :class="[
        'transition-all duration-300 ease-in-out overflow-hidden',
        collapsed ? 'max-h-0' : isCompact ? 'max-h-80' : 'max-h-[400px]'
      ]"
    >
      <div :class="['grid gap-3', isCompact ? 'px-3 pb-3 gap-2 grid-cols-2' : 'px-6 pb-6 grid-cols-2 gap-3']">
        <button
          v-for="(btn, index) in buttons"
          :key="btn.label"
          @click="btn.action"
          :disabled="isButtonDisabled(index)"
          :class="[
            btn.bgClass,
            'relative rounded-xl border border-game-border transition-all duration-200',
            'flex flex-col items-center justify-center text-center',
            isCompact
              ? 'min-h-[60px] py-2 px-1 gap-1'
              : 'min-h-[88px] p-4 gap-2',
            isButtonDisabled(index)
              ? 'opacity-40 cursor-not-allowed'
              : [btn.hoverClass, 'hover:scale-[1.02] hover:shadow-lg cursor-pointer active:scale-[0.98]'],
          ]"
        >
          <span :class="isCompact ? 'text-xl' : 'text-3xl'">{{ btn.icon }}</span>
          <span :class="['text-white font-semibold', isCompact ? 'text-xs' : 'text-sm']">{{ btn.label }}</span>
          <span v-if="!isCompact" class="text-gray-400 text-xs">{{ btn.description }}</span>
        </button>
      </div>
    </div>
  </div>
</template>
