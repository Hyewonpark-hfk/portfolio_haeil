<template>
  <div class="w-full">
    <!-- Full width Carousel on top -->
    <Carousel :slides="cardsForCarousel" autoplay :interval="5000" loop class="mb-8" />

    <!-- Full width list (stacked) -->
    <div ref="containerRef" class="w-full px-4 md:px-8">
      <div class="space-y-6">
        <a
          v-for="card in cards"
          :key="card.slug"
          :href="withBase(card.route)"
          class="group block w-full rounded-xl overflow-hidden border border-gray-300 bg-white shadow-sm 
                 transition-all duration-300 hover:shadow-lg hover:scale-[1.01] focus:outline-none focus:ring-2 
                 focus:ring-gray-400"
        >
          <div class="flex flex-col md:flex-row w-full">
            <div class="w-full md:w-1/3 h-56 md:h-auto flex-shrink-0">
              <img v-if="card.image" :src="card.image" alt="cover" class="w-full h-full object-cover" />
              <div v-else class="w-full h-full bg-gray-300 flex items-center justify-center text-gray-500">No Image</div>
            </div>
            <div class="p-4 md:p-6 flex-1">
              <h2 class="text-lg md:text-2xl font-semibold text-gray-900 mb-1">{{ card.title }}</h2>
              <h3 class="text-sm text-gray-600 mb-3">{{ card.name }}</h3>
              <p class="text-gray-700 text-sm leading-relaxed line-clamp-3 group-hover:line-clamp-none transition-all duration-300">{{ card.excerpt }}</p>
            </div>
          </div>
        </a>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { withBase } from 'vitepress'
import Carousel from './Carousel.vue'

type Card = {
  slug: string
  title: string
  name: string
  excerpt: string
  route: string      // `/works/?id=slug`
  image: string | null
}

// @ts-ignore - Vite's import.meta.glob is available at runtime
const markdownFiles = import.meta.glob('../../../works/**/index.md', {
  as: 'raw',
  eager: true,
})
// @ts-ignore - Vite's import.meta.glob is available at runtime
const imageFiles = import.meta.glob('../../../works/**/cover.{jpg,jpeg,png,webp}', {
  eager: true,
  import: 'default',
})

const cards = ref<Card[]>([])

for (const path in markdownFiles) {
  const raw = markdownFiles[path] as string
  const lines = raw.split('\n')

  const titleLine = lines.find(line => line.startsWith('# '))
  const nameLine = lines.find(line => line.startsWith('## '))
  const excerptLine = lines.find(line => line.trim() && !line.startsWith('#'))

  // docs/works/my-work/index.md -> slug = "my-work"
  const match = path.match(/works\/([^/]+)\/index\.md$/)
  const slug = match?.[1] ?? ''

  const route = `/works/?id=${slug}`

  const folder = path.replace(/\/index\.md$/, '/')
  const imageKey = Object.keys(imageFiles).find(k => k.startsWith(folder))

  cards.value.push({
    slug,
    title: titleLine?.replace(/^# /, '') || 'Untitled',
    name: nameLine?.replace(/^## /, '') || 'Anonymous',
    excerpt: excerptLine || '',
    route,
    image: imageKey ? (imageFiles[imageKey] as string) : null,
  })
}

// slides for carousel: map cards to slides and include href via withBase
const cardsForCarousel = computed(() =>
  cards.value.map(c => ({
    title: c.title,
    excerpt: c.excerpt,
    image: c.image,
    href: withBase(c.route),
  }))
)

const containerRef = ref<HTMLElement | null>(null)
</script>

<style scoped>
.line-clamp-3 {
  line-clamp: 3;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
