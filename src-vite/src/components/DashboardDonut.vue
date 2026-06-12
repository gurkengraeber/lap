<template>
  <div class="flex items-center gap-4">
    <svg width="110" height="110" viewBox="0 0 42 42" class="shrink-0 -rotate-90">
      <circle
        cx="21" cy="21" r="15.91549430918954"
        fill="transparent" stroke="var(--color-base-300)" stroke-width="6"
      />
      <circle
        v-for="(seg, i) in segments" :key="i"
        cx="21" cy="21" r="15.91549430918954"
        fill="transparent" :stroke="colors[i % colors.length]" stroke-width="6"
        :stroke-dasharray="`${seg.length} ${100 - seg.length}`"
        :stroke-dashoffset="-seg.offset"
      />
    </svg>
    <ul class="text-sm space-y-1 w-full min-w-0">
      <li v-for="(item, i) in items" :key="item.label" class="flex justify-between gap-2">
        <span class="flex items-center gap-2 truncate">
          <i class="w-2.5 h-2.5 rounded-full shrink-0 inline-block" :style="{ background: colors[i % colors.length] }"></i>
          <span class="truncate">{{ item.label }}</span>
        </span>
        <span class="text-base-content/60 shrink-0 tabular-nums">{{ item.count.toLocaleString() }} &middot; {{ formatFileSize(item.size) }}</span>
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { formatFileSize } from '@/common/utils'

const props = defineProps<{
  items: { label: string; count: number; size: number }[]
}>()

const colors = [
  '#3f8fce', '#5fb6f0', '#f0a35f', '#e0e0e0', '#9f7fe0', '#6ee0c0', '#e07f9f', '#c0c060',
]

const segments = computed(() => {
  const total = props.items.reduce((sum, item) => sum + item.count, 0)
  if (total === 0) return []

  let offset = 0
  return props.items.map(item => {
    const length = (item.count / total) * 100
    const seg = { length, offset }
    offset += length
    return seg
  })
})
</script>
