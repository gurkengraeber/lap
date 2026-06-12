<template>
  <div class="flex-1 overflow-y-auto p-4">
    <div v-if="loading" class="h-full flex items-center justify-center">
      <span class="loading loading-spinner loading-md text-primary"></span>
    </div>
    <div v-else-if="stats" class="max-w-5xl mx-auto flex flex-col gap-4 pb-4">
      <!-- header -->
      <div class="flex items-center justify-between flex-wrap gap-2">
        <h1 class="text-lg font-bold">{{ $t('dashboard.title') }}</h1>
        <div class="flex gap-6 bg-base-100 rounded-box px-4 py-2 border border-base-content/5">
          <div>
            <div class="text-xs text-base-content/50">{{ $t('dashboard.total_items') }}</div>
            <div class="text-lg font-bold tabular-nums">{{ stats.total_count.toLocaleString() }}</div>
          </div>
          <div>
            <div class="text-xs text-base-content/50">{{ $t('dashboard.total_size') }}</div>
            <div class="text-lg font-bold tabular-nums">{{ formatFileSize(stats.total_size) }}</div>
          </div>
        </div>
      </div>

      <!-- timeline heatmap -->
      <div class="rounded-box p-3 bg-base-300/30 border border-base-content/5 shadow-sm">
        <DashboardTimeline
          :timeline="stats.timeline"
          :years="stats.years"
          :year="year"
          @update:year="onYearChange"
        />
      </div>

      <!-- geo heatmap -->
      <div class="rounded-box p-3 bg-base-300/30 border border-base-content/5 shadow-sm">
        <h2 class="text-sm font-bold mb-2">{{ $t('dashboard.geo_heatmap') }}</h2>
        <DashboardGeoMap />
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <!-- file formats -->
        <div class="rounded-box p-3 bg-base-300/30 border border-base-content/5 shadow-sm">
          <h2 class="text-sm font-bold mb-2">{{ $t('dashboard.formats') }}</h2>
          <DashboardDonut v-if="stats.formats.length > 0" :items="formatItems" />
          <div v-else class="text-sm text-base-content/30 py-2">{{ $t('tooltip.not_found.data') }}</div>
        </div>

        <!-- storage usage -->
        <div class="rounded-box p-3 bg-base-300/30 border border-base-content/5 shadow-sm">
          <h2 class="text-sm font-bold mb-2">{{ $t('dashboard.storage_by_album') }}</h2>
          <DashboardBarList v-if="albumItems.length > 0" :items="albumItems" />
          <div v-else class="text-sm text-base-content/30 py-2">{{ $t('tooltip.not_found.data') }}</div>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <!-- top cameras -->
        <div class="rounded-box p-3 bg-base-300/30 border border-base-content/5 shadow-sm">
          <h2 class="text-sm font-bold mb-2">{{ $t('dashboard.top_cameras') }}</h2>
          <DashboardBarList v-if="cameraItems.length > 0" :items="cameraItems" color="var(--color-accent)" />
          <div v-else class="text-sm text-base-content/30 py-2">{{ $t('tooltip.not_found.data') }}</div>
        </div>

        <!-- top lenses -->
        <div class="rounded-box p-3 bg-base-300/30 border border-base-content/5 shadow-sm">
          <h2 class="text-sm font-bold mb-2">{{ $t('dashboard.top_lenses') }}</h2>
          <DashboardBarList v-if="lensItems.length > 0" :items="lensItems" color="var(--color-secondary)" />
          <div v-else class="text-sm text-base-content/30 py-2">{{ $t('tooltip.not_found.data') }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { getDashboardStats } from '@/common/api'
import { formatFileSize } from '@/common/utils'

import DashboardTimeline from '@/components/DashboardTimeline.vue'
import DashboardGeoMap from '@/components/DashboardGeoMap.vue'
import DashboardDonut from '@/components/DashboardDonut.vue'
import DashboardBarList from '@/components/DashboardBarList.vue'

const loading = ref(true)
const stats = ref<any>(null)
const year = ref(0)

onMounted(async () => {
  await loadStats(0)
})

async function onYearChange(newYear: number) {
  await loadStats(newYear)
}

async function loadStats(requestedYear: number) {
  loading.value = true
  const result = await getDashboardStats(requestedYear)
  if (result) {
    stats.value = result
    year.value = result.selected_year || requestedYear
  }
  loading.value = false
}

const formatItems = computed(() =>
  (stats.value?.formats || []).map((f: any) => ({ label: f.label, count: f.count, size: f.size }))
)

const albumItems = computed(() =>
  (stats.value?.albums || [])
    .filter((a: any) => a.name)
    .map((a: any) => ({ label: a.name, value: a.size, valueLabel: formatFileSize(a.size) }))
)

const cameraItems = computed(() =>
  (stats.value?.cameras || []).map((c: any) => ({
    label: [c.make, c.model].filter(Boolean).join(' '),
    value: c.count,
    valueLabel: c.count.toLocaleString(),
  }))
)

const lensItems = computed(() =>
  (stats.value?.lenses || []).map((l: any) => ({
    label: [l.make, l.model].filter(Boolean).join(' '),
    value: l.count,
    valueLabel: l.count.toLocaleString(),
  }))
)
</script>
