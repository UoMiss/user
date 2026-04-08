<template>
  <!-- Mobile Search + Filter -->
  <div class="lg:hidden mb-3 flex items-stretch gap-3">
    <div v-if="showSearch" class="category-mobile-search-wrap min-w-0 flex-1">
      <div class="category-search-pill">
        <svg class="category-search-pill-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-4.35-4.35m1.85-5.15a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
        <input :value="searchQuery" @input="$emit('update:searchQuery', ($event.target as HTMLInputElement).value)" type="text"
          class="min-w-0 flex-1 form-input category-search-pill-input"
          :placeholder="t('products.searchPlaceholder')" />
        <button v-if="searchQuery" type="button" @click="$emit('clearSearch')"
          :aria-label="t('common.cancel')"
          class="category-search-pill-clear">
          <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" aria-hidden="true">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.25" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>
    </div>

    <!-- Mobile Filter Button -->
    <button @click="$emit('update:showDrawer', true)"
      class="category-filter-trigger shrink-0 flex items-center justify-center gap-2 px-4 py-3 rounded-2xl text-sm font-medium min-h-[44px] whitespace-nowrap">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
          d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z" />
      </svg>
      {{ t('products.filter') }}
      <span v-if="selectedCategory" class="w-2 h-2 rounded-full theme-accent-stick"></span>
    </button>
  </div>

  <!-- Mobile Filter Drawer Overlay -->
  <Transition
    enter-active-class="transition duration-300 ease-out"
    enter-from-class="opacity-0"
    enter-to-class="opacity-100"
    leave-active-class="transition duration-200 ease-in"
    leave-from-class="opacity-100"
    leave-to-class="opacity-0">
    <div v-if="showDrawer" class="lg:hidden fixed inset-0 z-40 bg-black/50 cursor-pointer"
      @click="$emit('update:showDrawer', false)" style="overscroll-behavior: none;"></div>
  </Transition>

  <!-- Mobile Filter Drawer -->
  <Transition
    enter-active-class="transition duration-300 ease-out"
    enter-from-class="-translate-x-full"
    enter-to-class="translate-x-0"
    leave-active-class="transition duration-200 ease-in"
    leave-from-class="translate-x-0"
    leave-to-class="-translate-x-full">
    <div v-if="showDrawer"
      class="category-sidebar-shell lg:hidden fixed left-0 top-0 bottom-0 z-50 w-72 max-w-[80vw] backdrop-blur-xl border-r theme-border overflow-y-auto"
      style="overscroll-behavior: none;">
      <div class="p-6">
        <div class="flex items-center justify-between mb-6">
          <span class="text-sm font-bold theme-text-primary flex items-center gap-2">
            <span class="w-1 h-5 category-heading-accent rounded-full"></span>
            {{ showSearch ? t('products.filter') : t('products.categories') }}
          </span>
          <button @click="$emit('update:showDrawer', false)"
            class="category-mini-tool p-2 min-w-[44px] min-h-[44px] flex items-center justify-center rounded-xl">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>

        <!-- Category List -->
        <ul class="space-y-1">
          <li>
            <button @click="$emit('selectCategory', null, true)"
              class="category-item w-full text-left px-3 py-2 rounded-2xl transition-all duration-300 min-h-[44px]"
              :class="selectedCategory === null
                ? 'category-item-active'
                : 'category-item-idle theme-text-secondary hover:text-gray-900 dark:hover:text-white hover:bg-gray-50 dark:hover:bg-white/5'">
              {{ t('products.allCategories') }}
            </button>
          </li>
          <li v-for="group in categories" :key="group.id">
            <div class="space-y-1">
              <div class="flex items-stretch gap-1">
                <button @click="$emit('selectCategory', group.id, true)"
                  class="category-item flex-1 min-w-0 text-left px-3 py-2 rounded-2xl transition-all duration-300 flex items-center gap-2 min-h-[44px]"
                  :class="selectedCategory === group.id
                    ? 'category-item-active'
                    : 'category-item-idle theme-text-secondary hover:text-gray-900 dark:hover:text-white hover:bg-gray-50 dark:hover:bg-white/5'">
                  <img v-if="group.icon" :src="getImageUrl(group.icon)"
                    :alt="getLocalizedText(group.name)"
                    loading="lazy" class="h-5 w-5 rounded object-cover" />
                  <span class="truncate">{{ getLocalizedText(group.name) }}</span>
                </button>
                <button
                  v-if="group.children.length > 0"
                  type="button"
                  class="category-toggle h-10 w-10 shrink-0 self-center rounded-full flex items-center justify-center transition-all duration-300"
                  :class="expandedParentIds.includes(group.id) ? 'theme-text-primary' : 'theme-text-muted hover:text-gray-900 dark:hover:text-white'"
                  @click.stop="$emit('toggleParent', group.id)"
                >
                  <svg class="w-4 h-4 transition-transform duration-200"
                    :class="expandedParentIds.includes(group.id) ? 'rotate-90' : ''"
                    fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                  </svg>
                </button>
              </div>
              <ul v-if="group.children.length > 0 && expandedParentIds.includes(group.id)" class="space-y-1 pl-2.5">
                <li v-for="child in group.children" :key="child.id">
                  <button @click="$emit('selectCategory', child.id, true)"
                    class="category-item w-full text-left px-3 py-2 rounded-2xl transition-all duration-300 flex items-center gap-2 min-h-[44px]"
                    :class="selectedCategory === child.id
                      ? 'category-item-active'
                      : 'category-item-idle theme-text-secondary hover:text-gray-900 dark:hover:text-white hover:bg-gray-50 dark:hover:bg-white/5'">
                    <img v-if="child.icon" :src="getImageUrl(child.icon)"
                      :alt="getLocalizedText(child.name)"
                      loading="lazy" class="h-5 w-5 rounded object-cover" />
                    <span class="truncate">{{ getLocalizedText(child.name) }}</span>
                  </button>
                </li>
              </ul>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </Transition>

  <!-- Desktop Sidebar -->
  <aside class="hidden lg:block flex-shrink-0" :class="compact ? 'lg:w-60' : 'lg:w-64'">
    <div class="category-sidebar-shell backdrop-blur-xl border rounded-2xl sticky top-24" :class="compact ? 'p-5' : 'p-6'">
      <!-- Search (desktop, only for Products page) -->
      <div v-if="showSearch" :class="compact ? 'mb-4' : 'mb-6'">
        <div>
          <div class="category-search-pill">
            <svg class="category-search-pill-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-4.35-4.35m1.85-5.15a7 7 0 11-14 0 7 7 0 0114 0z" />
            </svg>
            <input :value="searchQuery" @input="$emit('update:searchQuery', ($event.target as HTMLInputElement).value)" type="text"
              class="min-w-0 flex-1 form-input category-search-pill-input"
              :placeholder="t('products.searchPlaceholder')" />
            <button v-if="searchQuery" type="button" @click="$emit('clearSearch')"
              :aria-label="t('common.cancel')"
              class="category-search-pill-clear">
              <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" aria-hidden="true">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.25" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
        </div>
      </div>

      <h2 :class="compact ? 'text-base font-bold mb-4' : 'text-lg font-bold mb-6'"
        class="theme-text-primary flex items-center gap-2">
        <span class="w-1 h-5 category-heading-accent rounded-full"></span>
        {{ t('products.categories') }}
      </h2>

      <ul :class="compact ? 'space-y-0.5' : 'space-y-1'">
        <li>
          <button @click="$emit('selectCategory', null)"
            class="category-item w-full text-left rounded-2xl transition-all duration-300"
            :class="[
              compact ? 'px-3 py-1.5 text-sm' : 'px-3 py-2',
              selectedCategory === null
                ? 'category-item-active'
                : 'category-item-idle theme-text-secondary hover:text-gray-900 dark:hover:text-white hover:bg-gray-50 dark:hover:bg-white/5'
            ]">
            {{ t('products.allCategories') }}
          </button>
        </li>
        <li v-for="group in categories" :key="group.id">
          <div :class="compact ? 'space-y-0.5' : 'space-y-1'">
            <div class="flex items-stretch" :class="compact ? 'gap-1' : 'gap-1.5'">
              <button @click="$emit('selectCategory', group.id)"
                class="category-item flex-1 min-w-0 text-left rounded-2xl transition-all duration-300 flex items-center gap-2"
                :class="[
                  compact ? 'px-3 py-1.5 text-sm' : 'px-3 py-2',
                  selectedCategory === group.id
                    ? 'category-item-active'
                    : 'category-item-idle theme-text-secondary hover:text-gray-900 dark:hover:text-white hover:bg-gray-50 dark:hover:bg-white/5'
                ]">
                <img v-if="group.icon" :src="getImageUrl(group.icon)"
                  :alt="getLocalizedText(group.name)"
                  loading="lazy" :class="compact ? 'h-4 w-4' : 'h-5 w-5'"
                  class="rounded object-cover" />
                <span class="truncate">{{ getLocalizedText(group.name) }}</span>
              </button>
              <button
                v-if="group.children.length > 0"
                type="button"
                class="category-toggle shrink-0 self-center rounded-full flex items-center justify-center transition-all duration-300"
                :class="[
                  compact ? 'h-9 w-9' : 'h-10 w-10',
                  expandedParentIds.includes(group.id) ? 'theme-text-primary' : 'theme-text-muted hover:text-gray-900 dark:hover:text-white'
                ]"
                @click.stop="$emit('toggleParent', group.id)"
              >
                <svg :class="[
                    compact ? 'w-3.5 h-3.5' : 'w-4 h-4',
                    expandedParentIds.includes(group.id) ? 'rotate-90' : ''
                  ]"
                  class="transition-transform duration-200"
                  fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                </svg>
              </button>
            </div>
            <ul v-if="group.children.length > 0 && expandedParentIds.includes(group.id)"
              :class="compact ? 'space-y-0.5 pl-2' : 'space-y-1 pl-2.5'">
              <li v-for="child in group.children" :key="child.id">
                <button @click="$emit('selectCategory', child.id)"
                  class="category-item w-full text-left rounded-2xl transition-all duration-300 flex items-center gap-2"
                  :class="[
                    compact ? 'px-3 py-1.5 text-sm' : 'px-3 py-2',
                    selectedCategory === child.id
                      ? 'category-item-active'
                      : 'category-item-idle theme-text-secondary hover:text-gray-900 dark:hover:text-white hover:bg-gray-50 dark:hover:bg-white/5'
                  ]">
                  <img v-if="child.icon" :src="getImageUrl(child.icon)"
                    :alt="getLocalizedText(child.name)"
                    loading="lazy" :class="compact ? 'h-4 w-4' : 'h-5 w-5'"
                    class="rounded object-cover" />
                  <span class="truncate">{{ getLocalizedText(child.name) }}</span>
                </button>
              </li>
            </ul>
          </div>
        </li>
      </ul>
    </div>
  </aside>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { getImageUrl } from '../utils/image'
import { useLocalized } from '../composables/useProduct'
import type { CategoryGroup } from '../utils/category'

const { t } = useI18n()
const { getLocalizedText } = useLocalized()

defineProps<{
  categories: CategoryGroup[]
  selectedCategory: number | null
  expandedParentIds: number[]
  showDrawer: boolean
  compact?: boolean
  showSearch?: boolean
  searchQuery?: string
}>()

defineEmits<{
  'selectCategory': [categoryId: number | null, closeDrawer?: boolean]
  'toggleParent': [categoryId: number]
  'update:showDrawer': [value: boolean]
  'update:searchQuery': [value: string]
  'clearSearch': []
}>()
</script>

<style scoped>
.category-filter-trigger {
  color: var(--ui-text-secondary);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 78%, white 22%),
      color-mix(in oklab, var(--ui-bg-soft) 80%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 88%, transparent);
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 18px 28px -22px color-mix(in oklab, black 24%, transparent);
  transition:
    transform 0.2s ease,
    color 0.2s ease,
    border-color 0.2s ease,
    box-shadow 0.2s ease;
}

.category-filter-trigger:hover {
  transform: translateY(-1px);
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 78%, white 22%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.24),
    0 20px 30px -22px color-mix(in oklab, black 28%, transparent);
}

.category-sidebar-shell {
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 82%, white 18%),
      color-mix(in oklab, var(--ui-bg-soft) 84%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 90%, transparent);
  border-color: color-mix(in oklab, var(--ui-border) 82%, white 18%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.18),
    0 22px 32px -28px color-mix(in oklab, black 24%, transparent);
}

.category-mini-tool {
  color: var(--ui-text-muted);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 78%, white 22%),
      color-mix(in oklab, var(--ui-bg-soft) 80%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 88%, transparent);
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 16px 24px -22px color-mix(in oklab, black 22%, transparent);
  transition:
    transform 0.2s ease,
    color 0.2s ease,
    border-color 0.2s ease,
    box-shadow 0.2s ease;
}

.category-mini-tool:hover {
  transform: translateY(-1px);
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 78%, white 22%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.24),
    0 18px 26px -22px color-mix(in oklab, black 26%, transparent);
}

.category-search-pill {
  position: relative;
  display: flex;
  align-items: center;
  flex: 1;
  min-width: 0;
  border: 1px solid color-mix(in oklab, var(--ui-border) 78%, white 22%);
  border-radius: 999px;
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 78%, white 22%),
      color-mix(in oklab, var(--ui-bg-soft) 80%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 88%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 14px 22px -24px color-mix(in oklab, black 18%, transparent);
  transition:
    border-color 0.2s ease,
    background-color 0.2s ease,
    box-shadow 0.2s ease;
}

.category-search-pill:focus-within {
  border-color: color-mix(in oklab, var(--ui-accent) 18%, transparent);
  background: color-mix(in oklab, var(--ui-bg-elevated) 94%, white 6%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.36),
    0 0 0 3px color-mix(in oklab, var(--ui-accent) 10%, transparent);
}

.category-search-pill-icon {
  position: absolute;
  left: 0.95rem;
  width: 0.9rem;
  height: 0.9rem;
  color: var(--ui-text-muted);
  pointer-events: none;
}

.category-search-pill-input {
  min-height: 2.5rem;
  padding-left: 2.65rem;
  padding-right: 2.8rem;
  border-radius: 999px;
  border-color: transparent !important;
  background: transparent !important;
  box-shadow: none !important;
}

.category-search-pill-input:focus {
  border-color: transparent !important;
  box-shadow: none !important;
}

.category-search-pill-clear {
  position: absolute;
  right: 0.55rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 1.55rem;
  height: 1.55rem;
  border-radius: 999px;
  color: color-mix(in oklab, var(--ui-text-muted) 92%, transparent);
  background: color-mix(in oklab, var(--ui-text-muted) 12%, transparent);
  transition:
    background-color 0.2s ease,
    color 0.2s ease,
    transform 0.2s ease;
}

.category-search-pill-clear:hover {
  color: var(--ui-text-primary);
  background: color-mix(in oklab, var(--ui-text-muted) 18%, transparent);
}

.category-item {
  box-shadow: none;
}

.category-item-idle {
  background: transparent;
}

.category-item-idle:hover {
  transform: translateX(1px);
  background: color-mix(in oklab, var(--ui-bg-soft) 68%, transparent);
}

.category-item-active {
  color: var(--ui-text-primary);
  background:
    linear-gradient(
      90deg,
      color-mix(in oklab, #ff3b30 12%, white 88%),
      color-mix(in oklab, var(--ui-bg-elevated) 60%, white 40%)
    );
  box-shadow:
    inset 2px 0 0 color-mix(in oklab, #ff3b30 72%, transparent),
    0 14px 22px -24px rgba(255, 59, 48, 0.1);
}

.category-item:focus-visible,
.category-toggle:focus-visible {
  outline: 2px solid color-mix(in oklab, var(--ui-accent) 22%, transparent);
  outline-offset: 2px;
}

.category-toggle {
  opacity: 0.78;
}

.category-toggle:hover {
  opacity: 1;
}

.category-heading-accent {
  background: color-mix(in oklab, #ff3b30 84%, transparent);
}

.category-search-pill-clear:focus-visible {
  outline: 2px solid color-mix(in oklab, var(--ui-accent) 24%, transparent);
  outline-offset: 2px;
}

:global(.dark .category-search-pill) {
  border-color: color-mix(in oklab, var(--ui-border) 74%, white 8%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 88%, white 12%),
      color-mix(in oklab, var(--ui-bg-soft) 86%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 16px 24px -24px rgba(0, 0, 0, 0.2);
}

:global(.dark .category-heading-accent) {
  background: color-mix(in oklab, var(--ui-accent) 72%, transparent);
}

:global(.dark .category-item) {
  box-shadow: none;
}

:global(.dark .category-item-idle) {
  background: transparent;
}

:global(.dark .category-item-idle:hover) {
  background: color-mix(in oklab, white 5%, transparent);
}

:global(.dark .category-item-active) {
  color: var(--ui-text-primary);
  background:
    linear-gradient(
      90deg,
      color-mix(in oklab, var(--ui-accent) 14%, transparent),
      color-mix(in oklab, var(--ui-bg-elevated) 18%, transparent)
    );
  box-shadow:
    inset 2px 0 0 color-mix(in oklab, var(--ui-accent) 54%, transparent),
    0 16px 24px -24px rgba(0, 0, 0, 0.18);
}

:global(.dark .category-search-pill:focus-within) {
  border-color: color-mix(in oklab, var(--ui-accent) 16%, transparent);
  background: color-mix(in oklab, var(--ui-bg-elevated) 86%, white 14%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.06),
    0 0 0 3px color-mix(in oklab, var(--ui-accent) 10%, transparent);
}

:global(.dark .category-filter-trigger) {
  color: var(--ui-text-secondary);
  border-color: color-mix(in oklab, var(--ui-border) 74%, white 8%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 88%, white 12%),
      color-mix(in oklab, var(--ui-bg-soft) 86%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 18px 28px -24px rgba(0, 0, 0, 0.22);
}

:global(.dark .category-filter-trigger:hover) {
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 74%, white 12%);
}

:global(.dark .category-sidebar-shell) {
  border-color: color-mix(in oklab, var(--ui-border) 74%, white 8%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 88%, white 12%),
      color-mix(in oklab, var(--ui-bg-soft) 86%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.05),
    0 24px 34px -28px rgba(0, 0, 0, 0.24);
}

:global(.dark .category-mini-tool) {
  color: var(--ui-text-muted);
  border-color: color-mix(in oklab, var(--ui-border) 74%, white 8%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 88%, white 12%),
      color-mix(in oklab, var(--ui-bg-soft) 86%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 16px 24px -24px rgba(0, 0, 0, 0.2);
}

:global(.dark .category-mini-tool:hover) {
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 74%, white 12%);
}

:global(.dark .category-search-pill-clear) {
  color: color-mix(in oklab, var(--ui-text-muted) 96%, transparent);
  background: color-mix(in oklab, white 10%, transparent);
}

:global(.dark .category-search-pill-clear:hover) {
  color: rgb(255 255 255);
  background: color-mix(in oklab, white 16%, transparent);
}

:global(.dark .category-toggle) {
  opacity: 0.72;
}

:global(.dark .category-toggle:hover) {
  opacity: 1;
}
</style>
