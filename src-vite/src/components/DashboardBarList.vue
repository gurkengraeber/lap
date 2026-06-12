<template>
  <div class="space-y-2 text-sm">
    <div v-for="item in items" :key="item.label">
      <div class="flex justify-between mb-1 gap-2">
        <span class="truncate">{{ item.label }}</span>
        <span class="text-base-content/60 shrink-0 tabular-nums">{{ item.valueLabel }}</span>
      </div>
      <div class="w-full bg-base-300 rounded-full h-2 overflow-hidden">
        <div
          class="h-2 rounded-full"
          :style="{ width: `${maxValue > 0 ? (item.value / maxValue) * 100 : 0}%`, background: color }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  items: { label: string; value: number; valueLabel: string }[]
  color?: string
}>()

const color = props.color || 'var(--color-primary)'

const maxValue = computed(() => props.items.reduce((max, item) => Math.max(max, item.value), 0))
</script>
