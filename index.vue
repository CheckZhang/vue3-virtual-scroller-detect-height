<script setup lang="ts">
import {
  computed,
  nextTick,
  onBeforeUnmount,
  onMounted,
  ref,
  watch,
} from 'vue';

interface Item {
  [key: string]: any;
  height?: number;
  index: number;
}

interface Props {
  bufferSize?: number;
  containerHeight?: number;
  focusPosition?: 'center' | 'end' | 'start';
  gap?: number;
  items: Item[];
  visibleItemsCount?: number;
}

const props = withDefaults(defineProps<Props>(), {
  containerHeight: 500,
  visibleItemsCount: 50,
  bufferSize: 5,
  gap: 0,
  focusPosition: 'start',
});

const emit = defineEmits<{
  mouseenter: [];
  scroll: [index: number];
}>();

const items1 = ref<Item[]>(props.items);

const firstVisibleIndex = ref(0);
const scrollContainer = ref<HTMLElement | null>(null);
let mutationObserver: MutationObserver | null = null;

const scrollToItem = async (index?: number) => {
  if (index === undefined) {
    index = firstVisibleIndex.value;
  } else {
    firstVisibleIndex.value = index;
    await nextTick();
  }
  await new Promise((resolve) => setTimeout(resolve, 100));

  if (!scrollContainer.value || index < 0 || index >= items1.value.length)
    return;

  // Calculate scroll position to show the target item at the top
  let targetScrollTop = 0;
  for (let i = 0; i < index; i++) {
    const item = items1.value[i];
    targetScrollTop += item?.height ?? 0;
  }

  // Set scroll position
  if (props.focusPosition === 'end')
    targetScrollTop -=
      props.containerHeight! - (items1.value[index]?.height ?? 0);
  else if (props.focusPosition === 'center')
    targetScrollTop -=
      props.containerHeight / 2 - (items1.value[index]?.height ?? 0) / 2 - 50;

  scrollContainer.value.scrollTop = targetScrollTop;
};

const measureItemHeight = (index: number) => {
  const el = scrollContainer.value?.querySelector<HTMLElement>(`#vs_${index}`);
  if (el) {
    const actualHeight = el.offsetHeight;
    if (items1.value[index] && items1.value[index].height !== actualHeight) {
      items1.value[index].height =
        actualHeight + (index === items1.value.length - 1 ? 0 : props.gap);
    }
  }
};

const measureItemHeights = () => {
  visibleItems.value.forEach((item) => measureItemHeight(item.index));
};

const proxyHeight = computed(() => {
  return items1.value.reduce((acc, item) => acc + (item.height || 0), 0);
});

const renderStartIndex = computed(() => {
  return Math.max(0, firstVisibleIndex.value - props.bufferSize);
});

const topSpacerHeight = computed(() => {
  let height = 0;
  for (let i = 0; i < renderStartIndex.value; i++) {
    height += items1.value[i]?.height ?? 0;
  }
  return height;
});

const visibleItems = computed(() => {
  const start = renderStartIndex.value;
  const end = Math.min(
    firstVisibleIndex.value + props.visibleItemsCount + props.bufferSize,
    items1.value.length,
  );
  return items1.value.slice(start, end);
});

const onScroll = () => {
  if (!scrollContainer.value) return;

  const scrollTop = scrollContainer.value.scrollTop;
  let cumulativeHeight = 0;
  let newFirstIndex = 0;

  for (const [i, element] of items1.value.entries()) {
    const itemHeight = element?.height ?? 0;
    if (cumulativeHeight + itemHeight > scrollTop) {
      newFirstIndex = i;
      break;
    }
    cumulativeHeight += itemHeight;
  }

  firstVisibleIndex.value = newFirstIndex;

  emit('scroll', firstVisibleIndex.value);
};

function getCenterItem(): Item | null {
  if (!scrollContainer.value) return null;

  const scrollTop = scrollContainer.value.scrollTop;
  const containerHeight = props.containerHeight || 0;
  const centerPosition = scrollTop + containerHeight / 2;

  let cumulativeHeight = 0;
  for (let i = 0; i < items1.value.length; i++) {
    const item = items1.value[i];
    const itemHeight = item?.height ?? 0;

    if (cumulativeHeight + itemHeight > centerPosition) {
      return item;
    }
    cumulativeHeight += itemHeight;
  }

  return items1.value[items1.value.length - 1] || null;
}

watch(
  () => props.items.length,
  () => {
    nextTick(() => {
      items1.value = props.items;
      measureItemHeights();
    });
  },
);

const api = {
  scrollToItem,
  firstVisibleIndex,
  renderStartIndex,
  visibleItems,
  measureItemHeight,
  getCenterItem,
  getScrollTop: () => scrollContainer.value?.scrollTop ?? 0,
};

defineExpose(api);

// Cleanup observer when component unmounts
const cleanup = () => {
  if (mutationObserver) {
    mutationObserver.disconnect();
    mutationObserver = null;
  }
  window.removeEventListener('resize', measureItemHeights);
};

onBeforeUnmount(cleanup);

onMounted(() => {
  window.addEventListener('resize', measureItemHeights);
});
</script>

<template>
  <div
    ref="scrollContainer"
    class="relative overflow-y-auto"
    :style="{
      height: `${containerHeight}px`,
    }"
    @scroll="onScroll"
    @mouseenter="emit('mouseenter')"
  >
    <div
      class="absolute top-0 w-1"
      :style="{ height: `${proxyHeight}px` }"
    ></div>
    <div
      class="relative flex flex-col"
      :style="{
        transform: `translateY(${topSpacerHeight}px)`,
        gap: `${gap}px`,
      }"
    >
      <div
        v-for="(item, index) in visibleItems"
        :key="item.index"
        :id="`vs_${item.index}`"
      >
        <slot :item="item" :index="index" :api="api"></slot>
      </div>
    </div>
  </div>
</template>
