# VirtualScrollByRender Component API Documentation

## Overview

A high-performance virtual scrolling component for Vue 3,it automatically measures actual hieght by real dom element, it renders only visible items to handle large datasets efficiently.

## Get Start

first, install the package:

```bash
npm install @vue3-virtual-scroller-detect-height
# or
yarn add @vue3-virtual-scroller-detect-height
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

![Virtual Scroll By Render Effect](./VirtualScrollByRender.gif)

```vue
<script setup lang="ts">
import { computed, nextTick, onMounted, ref } from 'vue';

import VirtualScrollBaseRender from 'vue3-virtual-scroller-detect-height';

const items = ref<Array<{ content: string; height: number; index: number }>>(
  [],
);
const virtualScrollRef = ref<InstanceType<
  typeof VirtualScrollBaseRender
> | null>(null);

const generateRandomText = () => {
  const words = [
    'Lorem',
    'ipsum',
    'dolor',
    'sit',
    'amet',
    'consectetur',
    'adipiscing',
    'elit',
    'sed',
    'do',
    'eiusmod',
    'tempor',
    'incididunt',
    'ut',
    'labore',
    'et',
    'dolore',
    'magna',
    'aliqua',
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
      height: 0,
      content: generateRandomText(),
    });
  }
};

const navigateToItem = async (index: number) => {
  virtualScrollRef.value?.scrollToItem(index);
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
  <div class="flex h-[700px] border">
    <div class="flex h-full w-64 flex-col border-r bg-gray-50 p-4">
      <div class="font-bold">Navigation</div>
      <div class="flex-grow overflow-y-auto">
        <div
          v-for="item in directoryItems"
          :key="`dd_${item.index}`"
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
        :container-height="700"
        :visible-items-count="50"
        :buffer-size="50"
        :gap="20"
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
    </div>
  </div>
</template>

<style scoped>
/* Add any custom styles if needed */
</style>

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
