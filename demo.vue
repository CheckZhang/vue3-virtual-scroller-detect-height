<script setup lang="ts">
import { computed, nextTick, onMounted, ref } from 'vue';

import { VirtualScrollBaseRender } from '@vben/elementplusplus';

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
