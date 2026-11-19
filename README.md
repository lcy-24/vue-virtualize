# vue-virtualize

Vue3 虚拟列表组件。**固定高度长列表只渲染可视区**，上万条数据也流畅，零依赖、全 TypeScript。

## 特性

- 🚀 **只渲染可视区**：DOM 节点数恒定（约视口条数 + 上下缓冲），与数据总量无关
- 🧭 **滚动缓冲（overscan）**：视口外预渲染少量项，快速滚动不白屏
- 🎯 **定位滚动**：`scrollToIndex`（支持 start/center/end）、`scrollToTop`、`scrollToBottom`
- 📐 **自适应视口**：`ResizeObserver` 监听容器尺寸变化，自动重算可见区间
- 🔑 **自定义 key**：可指定字段名做 key，避免 index 复用导致的组件状态错乱

## 安装

```bash
npm install vue-virtualize
```

> peer 依赖：`vue >= 3.3`。

## 快速开始

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { VirtualList } from 'vue-virtualize'

const list = ref(Array.from({ length: 100000 }, (_, i) => ({ id: i, text: `第 ${i} 条数据` })))
const listRef = ref()

function jump() {
  listRef.value?.scrollToIndex(50000, 'center')
}
</script>

<template>
  <button @click="jump">跳到第 50000 条</button>
  <VirtualList ref="listRef" :items="list" :item-size="40" height="400px" :buffer="8" key-field="id">
    <template #default="{ item, index }">
      <div style="height: 40px; line-height: 40px; border-bottom: 1px solid #eee">
        {{ index }}. {{ item.text }}
      </div>
    </template>
  </VirtualList>
</template>
```

## Props

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `items` | `any[]` | — | 数据源 |
| `itemSize` | `number` | — | 每项固定高度（px） |
| `height` | `number \| string` | `'400px'` | 容器高度 |
| `buffer` | `number` | `6` | 视口外预渲染项数 |
| `keyField` | `string` | `''` | 每项 key 字段名，默认用 index |

## 方法（通过 ref 调用）

| 方法 | 说明 |
| --- | --- |
| `scrollToIndex(index, align?)` | 滚动到指定索引，`align` 为 `start`/`center`/`end` |
| `scrollToTop()` | 回到顶部 |
| `scrollToBottom()` | 滚到底部 |

## 原理

外层一个可滚动容器，内部一个总高度为 `items.length × itemSize` 的撑高元素（spacer）；监听 `scroll` 事件，用 `scrollTop / itemSize` 算出起始索引，只渲染 `[start, end]` 区间的项，每项绝对定位到正确偏移。DOM 数量恒定，滚动性能与数据量解耦。

## License

MIT
