<script setup lang="ts">
import type { VideoFragment, MediaImageFragment } from '@@/types/shopify';
import type { EmblaCarouselType } from 'embla-carousel';

import emblaCarouselVue from 'embla-carousel-vue';

import { isMediaVideo, isMediaImage } from '@/helpers/shopify';

// Props
const props = defineProps<{
  productMedia: Array<VideoFragment | MediaImageFragment>;
}>();

// Embla setup
const [emblaRef, emblaApi] = emblaCarouselVue({ loop: true });

// State
const selectedIndex = ref<number>(0);
const canScrollNext = ref(false);
const canScrollPrev = ref(false);

// Next/prev actions
const scrollPrev = () => {
  emblaApi.value?.scrollPrev();
};

const scrollNext = () => {
  emblaApi.value?.scrollNext();
};

// ScrollTo method
const scrollTo = (index: number) => {
  emblaApi.value?.scrollTo(index);
};

// Embla event handlers
const onSelect = (api: EmblaCarouselType) => {
  canScrollNext.value = api?.canScrollNext() || false;
  canScrollPrev.value = api?.canScrollPrev() || false;
  selectedIndex.value = api?.selectedScrollSnap() || 0;
};

onMounted(() => {
  if (!emblaApi.value) {
    return;
  }

  emblaApi.value?.on('select', onSelect);
  emblaApi.value?.on('reInit', onSelect);
});
</script>

<template>
  <div ref="emblaRef" class="relative overflow-hidden lg:hidden">
    <div class="flex">
      <div
        v-for="(media, index) in props.productMedia"
        :key="media.id"
        class="flex-[0_0_100%] aspect-square"
      >
        <ShopifyVideo
          v-if="isMediaVideo(media)"
          :video="media"
        />
        <ShopifyImage
          v-else-if="isMediaImage(media)"
          :image="media.image"
          :alt="media.image?.altText || ''"
          :index="index"
        />
      </div>
    </div>
    <div class="absolute flex items-center justify-center gap-2 w-full p-2 bottom-0">
      <div v-for="(_, index) in productMedia" :key="index">
        <button
          class="size-2 rounded-full border border-black"
          :class="{ 'bg-black': index === selectedIndex }"
          @click="scrollTo(index)"
        />
      </div>
    </div>
    <button
      class="absolute flex items-center justify-center z-10 p-2 top-0 left-0 h-full"
      @click="scrollPrev"
    >
      <Icon name="ph:caret-left" class="size-5 shrink-0" />
    </button>
    <button
      class="absolute flex items-center justify-center z-10 p-2 top-0 right-0 h-full"
      @click="scrollNext"
    >
      <Icon name="ph:caret-right" class="size-5 shrink-0" />
    </button>
  </div>
</template>
