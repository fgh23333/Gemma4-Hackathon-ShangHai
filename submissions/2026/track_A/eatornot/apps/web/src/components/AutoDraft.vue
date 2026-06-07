<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Card, CardContent, CardHeader, CardTitle, Button } from '@/components/ui'
import { getUserId } from '@/api/client'

const props = defineProps<{ userId?: string }>()
const emit = defineEmits<{ 'confirm': [draft: any] }>()

const draft = ref<any>(null)

async function fetchDraft() {
  try {
    const res = await fetch(`/api/draft/auto?user_id=${props.userId || getUserId()}`)
    draft.value = await res.json()
  } catch {}
}

async function confirmDraft() {
  if (!draft.value) return
  emit('confirm', draft.value)
}

onMounted(fetchDraft)
</script>

<template>
  <Card v-if="draft">
    <CardHeader class="pb-2"><CardTitle class="text-base">🤖 智能推荐</CardTitle></CardHeader>
    <CardContent class="space-y-2 text-sm">
      <div v-for="item in draft.items || []" :key="item.name" class="flex justify-between">
        <span>{{ item.name }}</span>
        <span class="text-muted-foreground">¥{{ item.price }}</span>
      </div>
      <div v-if="draft.nutrition_summary" class="text-xs text-muted-foreground">{{ draft.nutrition_summary }}</div>
      <div class="flex gap-2">
        <Button size="sm" class="flex-1" @click="confirmDraft">确认</Button>
        <Button size="sm" variant="outline" @click="fetchDraft">🔄 换一个</Button>
      </div>
    </CardContent>
  </Card>
</template>
