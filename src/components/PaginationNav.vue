<template>
  <div v-if="totalPages > 1" class="flex justify-center" :class="compact ? 'mt-8' : 'mt-12'">
    <nav class="pagination-shell" :class="compact ? 'pagination-shell-compact' : 'pagination-shell-regular'">
      <button
        @click="$emit('changePage', currentPage - 1)"
        :disabled="currentPage === 1"
        :aria-label="`Previous page, page ${currentPage - 1}`"
        class="pagination-arrow disabled:opacity-30 disabled:cursor-not-allowed"
        :class="compact ? 'w-10 h-10' : 'w-12 h-12'"
      >
        <svg :class="compact ? 'w-4 h-4' : 'w-5 h-5'" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
        </svg>
      </button>

      <span class="pagination-status font-mono" :class="compact ? 'px-3 py-1.5 text-sm' : 'px-4 py-2.5'">
        <span class="pagination-status-current">{{ currentPage }}</span>
        <span :class="compact ? 'mx-1.5' : 'mx-2'" class="pagination-status-divider">/</span>
        <span class="pagination-status-total">{{ totalPages }}</span>
      </span>

      <button
        @click="$emit('changePage', currentPage + 1)"
        :disabled="currentPage === totalPages"
        :aria-label="`Next page, page ${currentPage + 1}`"
        class="pagination-arrow disabled:opacity-30 disabled:cursor-not-allowed"
        :class="compact ? 'w-10 h-10' : 'w-12 h-12'"
      >
        <svg :class="compact ? 'w-4 h-4' : 'w-5 h-5'" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
        </svg>
      </button>
    </nav>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  currentPage: number
  totalPages: number
  compact?: boolean
}>()

defineEmits<{
  'changePage': [page: number]
}>()
</script>

<style scoped>
.pagination-shell {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  border-radius: 1.35rem;
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 82%, white 18%),
      color-mix(in oklab, var(--ui-bg-soft) 84%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 90%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 20px 30px -26px color-mix(in oklab, black 24%, transparent);
}

.pagination-shell-compact {
  padding: 0.45rem;
}

.pagination-shell-regular {
  padding: 0.5rem;
}

.pagination-arrow {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  border-radius: 1rem;
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
    inset 0 1px 0 rgba(255, 255, 255, 0.18),
    0 14px 22px -22px color-mix(in oklab, black 22%, transparent);
  transition:
    transform 0.2s ease,
    color 0.2s ease,
    border-color 0.2s ease,
    box-shadow 0.2s ease;
}

.pagination-arrow:not(:disabled):hover {
  transform: translateY(-1px);
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 76%, white 24%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.24),
    0 16px 24px -22px color-mix(in oklab, black 24%, transparent);
}

.pagination-arrow:focus-visible {
  outline: none;
  box-shadow:
    0 0 0 3px var(--ui-focus-ring),
    0 16px 24px -22px color-mix(in oklab, black 24%, transparent);
}

.pagination-status {
  min-width: 5.4rem;
  text-align: center;
  border-radius: 1rem;
  border: 1px solid color-mix(in oklab, var(--ui-border) 78%, white 22%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 90%, white 10%),
      color-mix(in oklab, var(--ui-bg-soft) 76%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.16);
}

.pagination-status-current {
  color: var(--ui-text-primary);
  font-weight: 800;
}

.pagination-status-divider,
.pagination-status-total {
  color: var(--ui-text-muted);
}

:global(.dark .pagination-shell) {
  border-color: color-mix(in oklab, var(--ui-border) 76%, white 10%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 90%, white 10%),
      color-mix(in oklab, var(--ui-bg-soft) 86%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 22px 34px -28px rgba(0, 0, 0, 0.3);
}

:global(.dark .pagination-arrow) {
  border-color: color-mix(in oklab, var(--ui-border) 76%, white 10%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 88%, white 12%),
      color-mix(in oklab, var(--ui-bg-soft) 84%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 16px 24px -24px rgba(0, 0, 0, 0.26);
}

:global(.dark .pagination-arrow:not(:disabled):hover) {
  border-color: color-mix(in oklab, var(--ui-border-strong) 78%, white 12%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.06),
    0 18px 26px -24px rgba(0, 0, 0, 0.3);
}

:global(.dark .pagination-status) {
  border-color: color-mix(in oklab, var(--ui-border) 74%, white 10%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 90%, white 10%),
      color-mix(in oklab, var(--ui-bg-soft) 82%, transparent)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.04);
}
</style>
