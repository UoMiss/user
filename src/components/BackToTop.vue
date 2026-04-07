<template>
  <Transition
    enter-active-class="transition duration-300 ease-out"
    enter-from-class="opacity-0 scale-90"
    enter-to-class="opacity-100 scale-100"
    leave-active-class="transition duration-200 ease-in"
    leave-from-class="opacity-100 scale-100"
    leave-to-class="opacity-0 scale-90"
  >
    <button
      v-if="visible"
      @click="scrollToTop"
      class="back-to-top-button fixed right-4 lg:right-6 z-[45] flex h-11 w-11 items-center justify-center rounded-full border theme-btn-neutral shadow-lg transition-all hover:shadow-xl hover:-translate-y-0.5"
      :aria-label="t('common.backToTop')"
    >
      <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 15l7-7 7 7" />
      </svg>
    </button>
  </Transition>
</template>

<script setup lang="ts">
import { nextTick, onMounted, onUnmounted, ref, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute } from 'vue-router'

const { t } = useI18n()
const route = useRoute()
const visible = ref(false)

const onScroll = () => {
  visible.value = window.scrollY > 400
}

const scrollToTop = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const syncVisibilityAfterRouteChange = () => {
  window.requestAnimationFrame(() => {
    window.requestAnimationFrame(() => {
      onScroll()
    })
  })
}

onMounted(() => {
  onScroll()
  window.addEventListener('scroll', onScroll, { passive: true })
})

watch(
  () => route.fullPath,
  async () => {
    await nextTick()
    syncVisibilityAfterRouteChange()
  }
)

onUnmounted(() => window.removeEventListener('scroll', onScroll))
</script>

<style scoped>
.back-to-top-button {
  bottom: calc(5rem + env(safe-area-inset-bottom, 0px));
}

@media (min-width: 1024px) {
  .back-to-top-button {
    bottom: 1.5rem;
  }
}
</style>
