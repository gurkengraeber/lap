<template>
  <div>
    <div class="flex items-center justify-between mb-2">
      <h2 class="text-sm font-bold">{{ $t('dashboard.timeline') }}</h2>
      <div class="flex items-center gap-2 text-sm">
        <button
          class="px-1 rounded hover:bg-base-100 disabled:opacity-30 disabled:cursor-default cursor-pointer"
          :disabled="!hasPrevYear"
          @click="$emit('update:year', year - 1)"
        >
          <IconLeft class="w-4 h-4" />
        </button>
        <span class="font-mono tabular-nums">{{ year }}</span>
        <button
          class="px-1 rounded hover:bg-base-100 disabled:opacity-30 disabled:cursor-default cursor-pointer"
          :disabled="!hasNextYear"
          @click="$emit('update:year', year + 1)"
        >
          <IconRight class="w-4 h-4" />
        </button>
      </div>
    </div>

    <div v-if="weeks.length > 0" class="flex gap-[3px] overflow-x-auto pb-1">
      <div v-for="(week, wi) in weeks" :key="wi" class="flex flex-col gap-[3px]">
        <div
          v-for="(day, di) in week"
          :key="di"
          class="w-[11px] h-[11px] rounded-[2px]"
          :class="day ? levelClass(day.count) : 'invisible'"
          :title="day ? `${day.date}: ${day.count.toLocaleString()}` : ''"
        ></div>
      </div>
    </div>
    <div v-else class="text-sm text-base-content/30 py-2">
      {{ $t('tooltip.not_found.data') }}
    </div>

    <div class="flex items-center justify-end gap-1 mt-1 text-[10px] text-base-content/50">
      <span>{{ $t('dashboard.less') }}</span>
      <div class="w-[11px] h-[11px] rounded-[2px] level-0"></div>
      <div class="w-[11px] h-[11px] rounded-[2px] level-1"></div>
      <div class="w-[11px] h-[11px] rounded-[2px] level-2"></div>
      <div class="w-[11px] h-[11px] rounded-[2px] level-3"></div>
      <div class="w-[11px] h-[11px] rounded-[2px] level-4"></div>
      <span>{{ $t('dashboard.more') }}</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { IconLeft, IconRight } from '@/common/icons'

const props = defineProps<{
  timeline: { date: string; count: number }[]
  years: number[]
  year: number
}>()

defineEmits(['update:year'])

const hasPrevYear = computed(() => props.years.some(y => y < props.year))
const hasNextYear = computed(() => props.years.some(y => y > props.year))

const maxCount = computed(() =>
  props.timeline.reduce((max, d) => Math.max(max, d.count), 0)
)

// build a Monday-start week grid covering Jan 1 - Dec 31 of `year`
const weeks = computed(() => {
  if (!props.year) return []

  const countByDate = new Map(props.timeline.map(d => [d.date, d.count]))

  const start = new Date(props.year, 0, 1)
  const end = new Date(props.year, 11, 31)

  // shift so weeks start on Monday (getDay(): 0=Sun..6=Sat)
  const startOffset = (start.getDay() + 6) % 7

  const days: ({ date: string; count: number } | null)[] = []
  for (let i = 0; i < startOffset; i++) days.push(null)

  for (let d = new Date(start); d <= end; d.setDate(d.getDate() + 1)) {
    const y = d.getFullYear()
    const m = String(d.getMonth() + 1).padStart(2, '0')
    const day = String(d.getDate()).padStart(2, '0')
    const key = `${y}-${m}-${day}`
    days.push({ date: key, count: countByDate.get(key) || 0 })
  }

  const result: ({ date: string; count: number } | null)[][] = []
  for (let i = 0; i < days.length; i += 7) {
    result.push(days.slice(i, i + 7))
  }
  return result
})

function levelClass(count: number) {
  if (count <= 0) return 'level-0'
  const max = maxCount.value
  if (max <= 1) return 'level-4'
  const ratio = count / max
  if (ratio > 0.75) return 'level-4'
  if (ratio > 0.5) return 'level-3'
  if (ratio > 0.25) return 'level-2'
  return 'level-1'
}
</script>

<style scoped>
.level-0 { background-color: var(--color-base-300); }
.level-1 { background-color: color-mix(in srgb, var(--color-primary) 25%, var(--color-base-300)); }
.level-2 { background-color: color-mix(in srgb, var(--color-primary) 50%, var(--color-base-300)); }
.level-3 { background-color: color-mix(in srgb, var(--color-primary) 75%, var(--color-base-300)); }
.level-4 { background-color: var(--color-primary); }
</style>
