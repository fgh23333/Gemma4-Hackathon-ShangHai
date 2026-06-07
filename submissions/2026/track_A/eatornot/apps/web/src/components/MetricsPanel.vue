<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui'
import { getDemoMetrics, getUserId } from '@/api/client'

const props = defineProps<{ userId?: string }>()

const data = ref<any>(null)

async function load() {
  try {
    data.value = await getDemoMetrics(props.userId || getUserId())
  } catch {}
}

onMounted(load)
</script>

<template>
  <Card v-if="data">
    <CardHeader class="pb-2">
      <CardTitle class="text-base">📈 本周统计</CardTitle>
    </CardHeader>
    <CardContent class="text-sm space-y-2">
      <div v-for="m in (data.metrics || [])" :key="m.name" class="flex justify-between">
        <span>{{ m.name }}</span>
        <span class="font-medium">{{ m.value }}{{ m.unit || '' }}</span>
      </div>
      <div v-if="data.trend" class="text-xs text-muted-foreground">
        趋势: {{ data.trend }}
      </div>
    </CardContent>
  </Card>
  <div v-else class="text-xs text-muted-foreground py-2">暂无统计数据</div>
</template>
