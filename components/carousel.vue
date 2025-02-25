<script lang="ts" setup>
import { PropType } from 'vue';

const {
  images,
  slidingDuration,
  autoSlide,
  autoSlideDuration,
  autoSlideDirection,
  autoSlideDynamicDirection,
} = defineProps({
  images: {
    type: Array<string>,
    required: true
  },
  slidingDuration: {
    type: Number,
    default: 500,
  },
  autoSlide: {
    type: Boolean,
    default: true
  },
  autoSlideDuration: {
    type: Number,
    default: 5000
  },
  autoSlideDirection: {
    type: String as PropType<'left' | 'right'>,
    default: 'right',
    validator: (value: string) => ['left', 'right'].includes(value)
  },
  autoSlideDynamicDirection: {
    type: Boolean,
    default: false,
  }
})

const dynamicAutoSlideDirection = ref(autoSlideDirection);

const activeImageIndex = ref(0);
const isSliding = ref(false);

function setActiveImage(imageIndex: number) {
  if (isSliding.value) return;

  isSliding.value = true;

  activeImageIndex.value = imageIndex;

  setTimeout(() => {
    isSliding.value = false;
  }, slidingDuration);
}

function slideNext() {
  setActiveImage((activeImageIndex.value + 1) % images.length);

  if (autoSlideDynamicDirection) {
    dynamicAutoSlideDirection.value = 'right';
  }
}

function slidePrevious() {
  setActiveImage((activeImageIndex.value - 1 + images.length) % images.length);

  if (autoSlideDynamicDirection) {
    dynamicAutoSlideDirection.value = 'left';
  }
}

function isActiveImageByIndex(imageIndex: number) {
  return activeImageIndex.value === imageIndex;
}

onMounted(() => {
  if (autoSlide) {
    setInterval(() => {
      const direction = autoSlideDynamicDirection ? dynamicAutoSlideDirection.value : autoSlideDirection;
      if (direction === 'right') {
        slideNext();
      } else {
        slidePrevious();
      }
    }, autoSlideDuration);
  }
})
</script>

<template>
  <div class="relative w-full">
    <!-- Indicators -->
    <div
      class="absolute p-0 right-0 bottom-0 left-0 mb-4 flex justify-center gap-2 z-[2] box-border mx-[15%]"
    >
      <button
        v-for="(_imageUrl, imageIndex) in images"
        :key="imageIndex"
        @click="setActiveImage(imageIndex)"
        :class="[
          'w-7 h-1 border-y-[10px] box-content bg-white border-transparent bg-clip-padding',
          {
            'opacity-50 hover:scale-x-110 hover:opacity-75':
              !isActiveImageByIndex(imageIndex),
            'scale-y-125': isActiveImageByIndex(imageIndex),
          },
        ]"
      ></button>
    </div>

    <!-- Content -->
    <div class="relative w-full overflow-hidden">
      <div
        v-for="(imageUrl, imageIndex) in images"
        :key="imageIndex"
        :class="[
          'float-left w-full block relative -mr-[100%] inset-0',
          `transform transition-all duration-${slidingDuration} ease-in-out`,
          activeImageIndex > imageIndex
            ? '-translate-x-[100%]'
            : 'translate-x-[100%]',
          isActiveImageByIndex(imageIndex) ? 'translate-x-0' : '',
        ]"
      >
        <img :src="imageUrl" :alt="imageUrl" class="block w-full" />
      </div>
    </div>

    <!-- Prev button -->
    <button
      @click="slidePrevious()"
      class="absolute z-[1] inset-y-0 left-0 w-[15%] flex items-center justify-center group hover:bg-gradient-to-r hover:from-black/50 focus:outline-none"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="h-8 w-8 text-white opacity-50 group-hover:opacity-100 group-focus:opacity-100"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
        stroke-width="2"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M15 19l-7-7 7-7"
        />
      </svg>
    </button>
    <!-- Next button -->
    <button
      @click="slideNext()"
      class="absolute z-[1] inset-y-0 right-0 w-[15%] flex items-center justify-center group hover:bg-gradient-to-l hover:from-black/50 focus:outline-none"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="h-8 w-8 text-white opacity-50 group-hover:opacity-100 group-focus:opacity-100"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
        stroke-width="2"
      >
        <path stroke-linecap="round" stroke-linejoin="round" d="M9 5l7 7-7 7" />
      </svg>
    </button>
  </div>
</template>
