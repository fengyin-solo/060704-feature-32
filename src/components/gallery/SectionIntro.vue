<script setup lang="ts">
import { computed } from 'vue'
import type { GallerySection, Exhibit, DiaryState } from '@/types'
import { DiaryState as DS, STATE_NAMES } from '@/types'

interface Props {
  section: GallerySection | null
  exhibitCount: number
  exhibits?: Exhibit[]
}

const props = defineProps<Props>()

const hasExhibits = computed(() => props.exhibitCount > 0)

const exhibitList = computed(() => props.exhibits || [])

const stateStats = computed(() => {
  const stats: Record<DiaryState, number> = {
    [DS.SCHEDULED]: 0,
    [DS.FRESH]: 0,
    [DS.ROTTING]: 0,
    [DS.ROTTED]: 0,
    [DS.DYING]: 0,
    [DS.DEAD]: 0
  }
  exhibitList.value.forEach(e => {
    stats[e.diary.state]++
  })
  return stats
})

const freshCount = computed(() => stateStats.value[DS.FRESH])
const rottingCount = computed(() => stateStats.value[DS.ROTTING])
const rottedCount = computed(() => stateStats.value[DS.ROTTED] + stateStats.value[DS.DYING])

const uniqueAuthors = computed(() => {
  const authors = new Set(exhibitList.value.map(e => e.authorId))
  return authors.size
})

const recommendedHighlights = computed(() => {
  const highlights: string[] = []
  
  if (props.exhibitCount === 0) {
    return highlights
  }
  
  if (props.exhibitCount >= 10) {
    highlights.push(`🔥 本区人气旺盛，${props.exhibitCount} 件展品等你探索`)
  } else if (props.exhibitCount >= 5) {
    highlights.push(`✨ 本区精选 ${props.exhibitCount} 件作品，值得细细品味`)
  } else if (props.exhibitCount >= 3) {
    highlights.push(`📚 本区有 ${props.exhibitCount} 件展品，小巧精致`)
  } else {
    highlights.push(`💎 本区仅有 ${props.exhibitCount} 件展品，弥足珍贵`)
  }
  
  if (freshCount.value > 0) {
    highlights.push(`🆕 包含 ${freshCount.value} 件${STATE_NAMES[DS.FRESH]}作品`)
  }
  if (rottingCount.value > 0) {
    highlights.push(`⏳ ${rottingCount.value} 件作品正在${STATE_NAMES[DS.ROTTING]}，别错过`)
  }
  if (rottedCount.value > 0) {
    highlights.push(`🏺 ${rottedCount.value} 件深度腐化作品，别有韵味`)
  }
  
  if (uniqueAuthors.value > 1) {
    highlights.push(`👥 来自 ${uniqueAuthors.value} 位不同作者的创作`)
  }
  
  return highlights
})

const emptyGuideTips = computed(() => {
  const tips: { icon: string; text: string }[] = [
    { icon: '💡', text: '可以切换到其他展区看看，也许会有惊喜发现' },
    { icon: '🔄', text: '展品会随时间不断变化，过段时间再来可能就有新作品了' },
    { icon: '✍️', text: '你也可以发布自己的日记，让它在这里展出' }
  ]
  return tips
})
</script>

<template>
  <div v-if="section" class="bg-gradient-to-r from-gray-900 via-gray-800/50 to-gray-900 rounded-lg border border-gray-700 p-6 mb-6">
    <div class="flex items-start gap-4">
      <div class="text-5xl flex-shrink-0">{{ section.icon }}</div>
      <div class="flex-1">
        <div class="flex items-center gap-3 mb-2">
          <h3 class="font-vt323 text-2xl text-diary-fresh glow-text">
            {{ section.name }}
          </h3>
          <span 
            class="px-2 py-0.5 rounded text-xs font-vt323 border"
            :class="[
              hasExhibits 
                ? 'bg-diary-fresh/20 text-diary-fresh border-diary-fresh/50' 
                : 'bg-gray-800 text-gray-500 border-gray-700'
            ]"
          >
            {{ exhibitCount }} 件展品
          </span>
        </div>
        <p class="text-gray-400 font-vt323 text-sm leading-relaxed">
          {{ section.description }}
        </p>
        
        <div v-if="hasExhibits && recommendedHighlights.length > 0" class="mt-4 space-y-2">
          <div class="flex items-center gap-2 text-xs text-diary-fresh font-vt323">
            <span>🌟</span>
            <span>推荐看点</span>
          </div>
          <div class="flex flex-wrap gap-2">
            <span 
              v-for="(highlight, index) in recommendedHighlights" 
              :key="index"
              class="px-3 py-1 bg-gray-800/80 rounded text-xs font-vt323 text-gray-300 border border-gray-700"
            >
              {{ highlight }}
            </span>
          </div>
        </div>
        
        <div v-if="!hasExhibits" class="mt-4 space-y-3">
          <div class="flex items-center gap-2 text-xs text-gray-500 font-vt323">
            <span>🌱</span>
            <span>此展区暂无展品</span>
          </div>
          <div class="space-y-2">
            <p 
              v-for="(tip, index) in emptyGuideTips" 
              :key="index"
              class="text-xs font-vt323 text-gray-500 flex items-start gap-2"
            >
              <span class="flex-shrink-0">{{ tip.icon }}</span>
              <span>{{ tip.text }}</span>
            </p>
          </div>
        </div>
        
        <div v-if="hasExhibits" class="mt-3 flex items-center gap-2 text-xs text-gray-500 font-vt323">
          <span>📌</span>
          <span>点击展品卡片查看详情，或点击作者头像前往其日记墙</span>
        </div>
      </div>
    </div>
  </div>
</template>
