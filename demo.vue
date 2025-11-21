<script setup lang="ts">
import { computed, nextTick, onMounted, ref } from 'vue';

// for you, to import VirtualScrollBaseRender from 'vue3-virtual-scroller-detect-height';
import VirtualScrollBaseRender from './index.vue';

const items = ref<
  Array<{ content: string; height: number; image: string; index: number }>
>([]);

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
    const imgHeight = [200, 300, 500][Math.floor(Math.random() * 3)];
    items.value.push({
      index: i,
      height: 0,
      content: generateRandomText(),
      image:
        Math.random() > 0.8
          ? `https://picsum.photos/200/${imgHeight}?random=${Math.random()}`
          : '',
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
  <div
    style="
      min-height: 500px;
      height: 100%;
      display: flex;
      border: 1px solid #ccc;
      overflow: hidden;
    "
  >
    <div
      style="
        display: flex;
        height: 100%;
        width: 160px;
        flex-direction: column;
        border-right: 1px solid #ccc;
        background-color: #fff;
        padding: 16px;
      "
    >
      <div style="font-weight: bold">Navigation</div>
      <div style="flex-grow: 1; overflow-y: auto">
        <div
          v-for="item in directoryItems"
          :key="`dd_${item.index}`"
          @click="navigateToItem(item.index)"
          style="cursor: pointer; border-radius: 5px; padding: 8px"
        >
          {{ item.title }}
        </div>
      </div>
    </div>

    <div style="flex: 1; height: 100%">
      <VirtualScrollBaseRender
        ref="virtualScrollRef"
        :items="items"
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
                  // You can do any async thing for the element, like render element to markdown by some third package
                  await nextTick();
                  api.measureItemHeight(item.index);
                }
              }
            "
          >
            <div class="mb-2 text-lg font-semibold">
              Item {{ item.index + 1 }}
            </div>
            <img
              v-if="item.image"
              :src="item.image"
              @load="() => api.measureItemHeight(item.index)"
            />
            <div class="leading-relaxed text-gray-600">{{ item.content }}</div>
          </div>
        </template>
      </VirtualScrollBaseRender>
    </div>
  </div>
</template>
