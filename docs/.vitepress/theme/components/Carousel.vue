<template>
  <div
    class="relative w-full overflow-hidden rounded-2xl bg-black text-white"
    @mouseenter="pause()"
    @mouseleave="resume()"
  >
    <!-- Slides -->
    <div class="relative h-64 md:h-96">
      <template v-for="(slide, i) in slides" :key="i">
        <a
          v-if="slide.href"
          :href="slide.href"
          class="absolute inset-0 transition-opacity duration-500"
          :class="currentIndex === i ? 'opacity-100 pointer-events-auto' : 'opacity-0 pointer-events-none'"
          >
          <img
            v-if="slide.image"
            :src="slide.image"
            alt="slide image"
            class="w-full h-full object-cover"
          />
          <div
            class="absolute inset-0 bg-black/40 flex items-end md:items-center md:justify-start p-6 md:p-12"
          >
            <div class="max-w-2xl">
              <h3 class="text-xl md:text-3xl font-semibold">{{ slide.title }}</h3>
              <p class="text-sm md:text-base text-gray-200">{{ slide.excerpt }}</p>
            </div>
          </div>
        </a>

        <div
          v-else
          class="absolute inset-0 transition-opacity duration-500"
          :class="currentIndex === i ? 'opacity-100 pointer-events-auto' : 'opacity-0 pointer-events-none'"
        >
          <img
            v-if="slide.image"
            :src="slide.image"
            alt="slide image"
            class="w-full h-full object-cover"
          />
          <div class="absolute inset-0 bg-black/40 flex items-end md:items-center md:justify-start p-6 md:p-12">
            <div class="max-w-2xl">
              <h3 class="text-xl md:text-3xl font-semibold">{{ slide.title }}</h3>
              <p class="text-sm md:text-base text-gray-200">{{ slide.excerpt }}</p>
            </div>
          </div>
        </div>
      </template>
    </div>

    <!-- Prev / Next arrows -->
    <button
      class="absolute left-2 top-1/2 -translate-y-1/2 bg-black/50 hover:bg-black/70 text-white rounded-full p-2 md:p-3"
      @click="prev"
      aria-label="Previous"
    >
      ‹
    </button>
    <button
      class="absolute right-2 top-1/2 -translate-y-1/2 bg-black/50 hover:bg-black/70 text-white rounded-full p-2 md:p-3"
      @click="next"
      aria-label="Next"
    >
      ›
    </button>

    <!-- Mobile tap areas -->
    <div class="md:hidden absolute inset-0 flex">
      <div class="w-1/2 h-full" @click="prev" />
      <div class="w-1/2 h-full" @click="next" />
    </div>

    <!-- Dots -->
    <div class="absolute bottom-4 left-1/2 -translate-x-1/2 flex items-center gap-2">
      <button
        v-for="(slide, i) in slides"
        :key="i"
        @click="goTo(i)"
        :class="['w-3 h-3 rounded-full', currentIndex === i ? 'bg-white' : 'bg-white/40']"
        aria-label="Go to slide"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount, toRef } from 'vue'

type Slide = {
  title?: string
  excerpt?: string
  image?: string | null
  href?: string | null
}

const props = defineProps<{
  slides: Slide[]
  autoplay?: boolean
  interval?: number
  loop?: boolean
}>()

const slides = toRef(props, 'slides')
const autoplay = props.autoplay ?? false
const interval = props.interval ?? 5000
const loop = props.loop ?? true

const currentIndex = ref(0)
const paused = ref(false)
let timer: ReturnType<typeof setInterval> | null = null

function startTimer() {
  if (!autoplay) return
  clearTimer()
  timer = setInterval(() => {
    if (paused.value) return
    next()
  }, interval)
}

function clearTimer() {
  if (timer) {
    clearInterval(timer)
    timer = null
  }
}

function prev() {
  if (currentIndex.value > 0) currentIndex.value--
  else if (loop) currentIndex.value = Math.max(0, slides.value.length - 1)
}

function next() {
  if (currentIndex.value < slides.value.length - 1) currentIndex.value++
  else if (loop) currentIndex.value = 0
}

function goTo(i: number) {
  currentIndex.value = i
}

function pause() {
  paused.value = true
}

function resume() {
  paused.value = false
}

watch(slides, () => {
  // clamp currentIndex if slides change
  if (currentIndex.value >= slides.value.length) currentIndex.value = 0
})

onMounted(() => startTimer())
onBeforeUnmount(() => clearTimer())
</script>

<style scoped>
.carousel-enter-active, .carousel-leave-active {
  transition: opacity 0.5s ease;
}
.carousel-enter-from, .carousel-leave-to {
  opacity: 0;
}
</style>
