# VirtualScrollByRender Component API Documentation

## Overview
A high-performance virtual scrolling component for Vue 3 that renders only visible items to handle large datasets efficiently.

## Get Start

first, install the package:

```bash
npm install @vben/elementplusplus
# or
yarn add @vben/elementplusplus
```

Then import and use the component in your Vue 3 application:

```vue
import { VirtualScrollBaseRender } from '@vben/elementplusplus';
```

Then you can use the `VirtualScrollBaseRender` component in your templates.
```vue
      <VirtualScrollBaseRender
        :items="items"
        :container-height="700"
      >
        <template #default="{ item, api }">
          <div
            class="border-b border-gray-300 bg-white p-4"
            :ref="
              async (ele) => {
                if (ele) {
                  await nextTick();
                  api.measureItemHeight(item.index);
                }
              }
            "
          >
            <div class="mb-2 text-lg font-semibold">
              Item {{ item.index + 1 }}
            </div>
            <div class="leading-relaxed text-gray-600">{{ item.content }}</div>
          </div>
        </template>
      </VirtualScrollBaseRender>
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `items` | `Item[]` | **Required** | Array of items to render. Each item must have an `index` property |
| `containerHeight` | `number` | `500` | Height of the scroll container in pixels |
| `visibleItemsCount` | `number` | `50` | Number of items to render in viewport |
| `estimatedItemHeight` | `number` | `80` | Estimated height for items before measurement |
| `bufferSize` | `number` | `5` | Number of items to render outside viewport for smooth scrolling |

## Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `scroll` | `(index: number)` | Emitted when scroll position changes |
| `remove` | `(index: number, id: string)` | Emitted when an item is removed from DOM |
| `mouseenter` | `()` | Emitted when mouse enters the container |

## Exposed Methods

| Method | Parameters | Description |
|--------|------------|-------------|
| `scrollToItem` | `(index: number)` | Scrolls to show the specified item at the top |

## Exposed Properties

| Property | Type | Description |
|----------|------|-------------|
| `firstVisibleIndex` | `Ref<number>` | Index of the first visible item |
| `renderStartIndex` | `Ref<number>` | Index where rendering starts (includes buffer) |
| `visibleItems` | `Ref<Item[]>` | Currently rendered items |

## Slots

### Default Slot
```vue
<template #default="{ item, index }">
  <!-- Your item template -->
</template>
```

**Slot Props:**
- `item`: The current item object
- `index`: The item's position in the visible items array

## Example Usage

```vue
<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';
import { VirtualScrollBaseRender } from '@vben/elementplusplus';

const items = ref<Array<{ content: string; height: number; index: number }>>([]);
const virtualScrollRef = ref<InstanceType<typeof VirtualScrollBaseRender> | null>(null);
const showDirectory = ref(false);

const generateRandomText = () => {
  const words = [
    'Lorem', 'ipsum', 'dolor', 'sit', 'amet', 'consectetur', 'adipiscing', 'elit',
    'sed', 'do', 'eiusmod', 'tempor', 'incididunt', 'ut', 'labore', 'et', 'dolore', 'magna', 'aliqua',
  ];
  const length = Math.floor(Math.random() * 200) + 20;
  let text = '';
  for (let i = 0; i < length; i++) {
    text += `${words[Math.floor(Math.random() * words.length)]} `;
  }
  return text.trim();
};

const generateItems = () => {
  for (let i = 0; i < 1000; i++) {
    items.value.push({
      index: i,
      height: 80,
      content: generateRandomText(),
    });
  }
};

const navigateToItem = (index: number) => {
  virtualScrollRef.value?.scrollToItem(index);
  showDirectory.value = false;
};

const directoryItems = computed(() => {
  return items.value
    .filter((_, index) => index % 50 === 0)
    .map((item) => ({
      index: item.index,
      title: `Item ${item.index + 1}`,
    }));
});

onMounted(() => {
  generateItems();
});
</script>

<template>
  <div class="flex h-screen">
    <div class="w-64 border-r bg-gray-50 p-4">
      <button
        @click="showDirectory = !showDirectory"
        class="mb-4 w-full rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600"
      >
        {{ showDirectory ? 'Hide' : 'Show' }} Directory
      </button>

      <div v-if="showDirectory" class="max-h-96 overflow-y-auto">
        <div
          v-for="item in directoryItems"
          :key="item.index"
          @click="navigateToItem(item.index)"
          class="cursor-pointer rounded p-2 hover:bg-blue-100"
        >
          {{ item.title }}
        </div>
      </div>
    </div>

    <div class="flex-1">
      <VirtualScrollBaseRender
        ref="virtualScrollRef"
        :items="items"
        :container-height="500"
        :visible-items-count="50"
        :buffer-size="50"
      >
        <template #default="{ item }">
          <div class="border-b border-gray-300 bg-white p-4">
            <div class="mb-2 text-lg font-semibold">
              Item {{ item.index + 1 }}
            </div>
            <div class="leading-relaxed text-gray-600">{{ item.content }}</div>
          </div>
        </template>
      </VirtualScrollBaseRender>
    </div>
  </div>
</template>
```

## Features

- **Dynamic Height Support**: Automatically measures and adjusts to actual item heights
- **Smooth Scrolling**: Buffer zones prevent blank content during fast scrolling
- **Navigation Support**: Programmatic scrolling to specific items
- **Performance Optimized**: Only renders visible items + buffer
- **TypeScript Support**: Full type safety with proper interfaces

## Notes

- Items must have a unique `index` property
- The component uses `transform: translateY()` for optimal performance
- Height measurements are performed after DOM insertion
- Buffer zones help maintain smooth scrolling experience
