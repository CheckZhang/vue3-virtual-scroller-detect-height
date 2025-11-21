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
  containerHeight?: number | string;
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
  scroll: [index: number];
}>();

const items1 = ref<Item[]>(props.items);
const viewWidth = ref(500);
const viewHeight = ref(500);

const firstVisibleIndex = ref(0);
const scrollContainer = ref<HTMLElement | null>(null);
const rootContainer = ref<HTMLElement | null>(null);
let observer: ResizeObserver;

const startObserving = () => {
  observer = new ResizeObserver((entries: ResizeObserverEntry[]) => {
    const entry = entries[0];
    if (entry) {
      const { width, height } = entry.contentRect;
      viewWidth.value = width;
      viewHeight.value = height;
      measureItemHeights();
    }
  });
  observer.observe(rootContainer.value!);
};

const stopObserving = () => {
  observer?.unobserve(rootContainer.value!);
  observer?.disconnect();
};

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

  let targetScrollTop = 0;
  for (let i = 0; i < index; i++) {
    const item = items1.value[i];
    targetScrollTop += item?.height ?? 0;
  }

  if (props.focusPosition === 'end')
    targetScrollTop -= viewHeight.value - (items1.value[index]?.height ?? 0);
  else if (props.focusPosition === 'center')
    targetScrollTop -=
      viewHeight.value / 2 - (items1.value[index]?.height ?? 0) / 2 - 50;

  scrollContainer.value.scrollTo({
    top: targetScrollTop,
    behavior: 'smooth',
  });
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

function getCenterItem(): Item | undefined {
  if (!scrollContainer.value) return;

  const scrollTop = scrollContainer.value.scrollTop;
  const centerPosition = scrollTop + viewHeight.value / 2;

  let cumulativeHeight = 0;
  for (let i = 0; i < items1.value.length; i++) {
    const item = items1.value[i];
    const itemHeight = item?.height ?? 0;

    if (cumulativeHeight + itemHeight > centerPosition) {
      return item;
    }
    cumulativeHeight += itemHeight;
  }

  return items1.value[items1.value.length - 1];
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

onBeforeUnmount(() => {
  stopObserving();
});

onMounted(() => {
  startObserving();
  measureItemHeights();
});
</script>

<template>
  <div
    ref="rootContainer"
    v-bind="$attrs"
    style="height: 100%; overflow: hidden"
  >
    <div
      ref="scrollContainer"
      style="position: relative; overflow-y: auto; min-height: 0"
      :style="{ height: `${viewHeight}px` }"
      @scroll="onScroll"
    >
      <div
        style="position: absolute; top: 0; width: 8px"
        :style="{ height: `${proxyHeight}px` }"
      ></div>
      <div
        style="display: flex; flex-direction: column; min-height: 0"
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
  </div>
</template>
