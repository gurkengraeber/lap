<template>
  <div class="relative w-full h-[260px] rounded-box overflow-hidden border border-base-content/5">
    <div v-if="loading" class="absolute inset-0 flex items-center justify-center z-50 bg-base-200/50">
      <span class="loading loading-spinner loading-md text-primary"></span>
    </div>
    <div ref="mapEl" class="w-full h-full"></div>
    <div
      v-if="!loading && points.length === 0"
      class="absolute inset-0 flex items-center justify-center text-base-content/30 px-4"
    >
      <span class="text-sm text-center">{{ $t('tooltip.not_found.location_hint') }}</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { config } from '@/common/config'
import { getGpsHeatmapPoints } from '@/common/api'

import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import 'leaflet.heat'

const mapEl = ref(null)
const loading = ref(true)
const points = ref([])

let map = null
let tileLayer = null
let heatLayer = null
let resizeObserver = null
let tileErrorFallbackTriggered = false

const mapThemes = [
  {
    url: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
    attribution: 'OpenStreetMap',
  },
  {
    url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
    attribution: 'Powered by Esri',
  },
]

onMounted(async () => {
  map = L.map(mapEl.value, {
    center: [20, 0],
    zoom: 2,
    keyboard: false,
    zoomControl: false,
    attributionControl: false,
    dragging: false,
    scrollWheelZoom: false,
    doubleClickZoom: false,
    boxZoom: false,
    touchZoom: false,
  })

  resizeObserver = new ResizeObserver(() => {
    if (map) map.invalidateSize()
  })
  resizeObserver.observe(mapEl.value.parentElement)

  updateTheme()

  const gridPoints = await getGpsHeatmapPoints()
  points.value = gridPoints
  loading.value = false

  if (gridPoints.length > 0) {
    const weights = gridPoints.map(p => Math.log1p(p.count))
    const maxWeight = Math.max(...weights)
    const heatPoints = gridPoints.map((p, i) => [p.lat, p.lon, weights[i] / maxWeight])

    heatLayer = L.heatLayer(heatPoints, {
      radius: 14,
      blur: 10,
      maxZoom: 17,
      max: 1.0,
      minOpacity: 0.35,
      gradient: { 0.2: '#1d4ed8', 0.4: '#06b6d4', 0.6: '#22c55e', 0.8: '#facc15', 1.0: '#ef4444' },
    }).addTo(map)

    const latLngs = gridPoints.map(p => [p.lat, p.lon])
    map.fitBounds(L.latLngBounds(latLngs), { padding: [20, 20] })
  }
})

onBeforeUnmount(() => {
  if (map) map.remove()
  if (resizeObserver) resizeObserver.disconnect()
})

function updateTheme() {
  const theme = mapThemes[Number(config.infoPanel.mapTheme)] || mapThemes[0]
  if (!map) return
  if (tileLayer) {
    map.removeLayer(tileLayer)
    tileLayer = null
  }
  tileErrorFallbackTriggered = false
  tileLayer = L.tileLayer(theme.url, { attribution: theme.attribution }).addTo(map)
  if (heatLayer) heatLayer.bringToFront()
  tileLayer.on('tileerror', () => {
    if (tileErrorFallbackTriggered) return
    tileErrorFallbackTriggered = true
    if (Number(config.infoPanel.mapTheme) !== 0) {
      config.infoPanel.mapTheme = 0
      updateTheme()
    }
  })
}
</script>
