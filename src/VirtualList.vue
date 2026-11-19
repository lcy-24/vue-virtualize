<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

interface Props {
  /** 数据源 */
  items: any[]
  /** 每一项的固定高度（px） */
  itemSize: number
  /** 容器高度，支持数字(px)或 CSS 字符串，默认 400px */
  height?: number | string
  /** 视口外多渲染的项数（滚动缓冲，减少白屏），默认 6 */
  buffer?: number
  /** 指定每项唯一 key 的字段名，默认用 index */
  keyField?: string
}

const props = withDefaults(defineProps<Props>(), {
  height: '400px',
  buffer: 6,
  keyField: '',
})

const emit = defineEmits<{
  (e: 'scroll', event: Event): void
}>()

const containerRef = ref<HTMLElement>()
const scrollTop = ref(0)
const viewportHeight = ref(0)

const totalHeight = computed(() => props.items.length * props.itemSize)

const startIndex = computed(() =>
  Math.max(0, Math.floor(scrollTop.value / props.itemSize) - props.buffer)
)

const endIndex = computed(() =>
  Math.min(
    props.items.length,
    Math.ceil((scrollTop.value + viewportHeight.value) / props.itemSize) + props.buffer
  )
)

interface VisibleItem {
  item: any
  index: number
  style: Record<string, string>
}

const visibleItems = computed<VisibleItem[]>(() => {
  const start = startIndex.value
  const end = endIndex.value
  const result: VisibleItem[] = []
  for (let i = start; i < end; i++) {
    result.push({
      item: props.items[i],
      index: i,
      style: {
        position: 'absolute',
        top: `${i * props.itemSize}px`,
        height: `${props.itemSize}px`,
        left: '0',
        right: '0',
      },
    })
  }
  return result
})

const containerStyle = computed(() => ({
  height: typeof props.height === 'number' ? `${props.height}px` : props.height,
}))

function onScroll(event: Event) {
  scrollTop.value = (event.target as HTMLElement).scrollTop
  emit('scroll', event)
}

/** 滚动到指定索引 */
function scrollToIndex(index: number, align: 'start' | 'center' | 'end' = 'start') {
  const el = containerRef.value
  if (!el) return
  const maxIndex = props.items.length - 1
  const target = Math.min(Math.max(index, 0), maxIndex)
  let top = target * props.itemSize
  if (align === 'center') top -= (el.clientHeight - props.itemSize) / 2
  else if (align === 'end') top -= el.clientHeight - props.itemSize
  el.scrollTop = Math.max(0, top)
}

function scrollToTop() {
  if (containerRef.value) containerRef.value.scrollTop = 0
}

function scrollToBottom() {
  if (containerRef.value) containerRef.value.scrollTop = containerRef.value.scrollHeight
}

function updateViewport() {
  viewportHeight.value = containerRef.value?.clientHeight ?? 0
}

let resizeObserver: ResizeObserver | null = null

onMounted(() => {
  updateViewport()
  if (typeof ResizeObserver !== 'undefined' && containerRef.value) {
    resizeObserver = new ResizeObserver(updateViewport)
    resizeObserver.observe(containerRef.value)
  }
})

onBeforeUnmount(() => {
  resizeObserver?.disconnect()
})

defineExpose({ scrollToIndex, scrollToTop, scrollToBottom })
</script>

<template>
  <div
    ref="containerRef"
    class="vv-list"
    :style="containerStyle"
    @scroll="onScroll"
  >
    <div
      class="vv-list__spacer"
      :style="{ height: totalHeight + 'px', position: 'relative', width: '100%' }"
    >
      <div
        v-for="vi in visibleItems"
        :key="keyField ? vi.item[keyField] : vi.index"
        class="vv-list__item"
        :style="vi.style"
      >
        <slot :item="vi.item" :index="vi.index" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.vv-list {
  position: relative;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

.vv-list__item {
  box-sizing: border-box;
}
</style>
