<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Card, CardContent, CardHeader, CardTitle, Badge } from '@/components/ui'
import { getLearningPoints, getUserId } from '@/api/client'

const props = defineProps<{ userId?: string }>()

const data = ref<any>(null)

async function load() {
  try {
    data.value = await getLearningPoints(props.userId || getUserId())
  } catch {}
}

onMounted(load)
</script>

<template>
  <Card v-if="data">
    <CardHeader class="pb-2">
      <CardTitle class="text-base">🧠 学习记录</CardTitle>
    </CardHeader>
    <CardContent class="space-y-2 text-sm">
      <div v-for="pt in (data.points || [])" :key="pt.id" class="p-2 rounded bg-gray-50">
        <div class="flex items-center justify-between">
          <span class="font-medium">{{ pt.title }}</span>
          <Badge variant="secondary" class="text-xs">{{ pt.category }}</Badge>
        </div>
        <p class="text-xs text-muted-foreground mt-1">{{ pt.summary }}</p>
      </div>
      <div v-if="data.summary" class="text-xs text-muted-foreground">
        共 {{ data.points?.length || 0 }} 条学习记录
      </div>
    </CardContent>
  </Card>
  <div v-else class="text-xs text-muted-foreground py-2">暂无学习记录</div>
</template>
