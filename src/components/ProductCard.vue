<template>
  <div
    class="product-card-shell group relative flex h-full cursor-pointer flex-col overflow-hidden rounded-[1.5rem] transition-all duration-300 theme-slide-up"
    :class="isSoldOut(product)
      ? 'product-card-shell-sold opacity-60 grayscale-[0.12] saturate-[0.82] hover:-translate-y-0 hover:shadow-none'
      : 'product-card-shell-active hover:-translate-y-1.5'"
    :style="{ animationDelay: `${index * animationStep}ms` }"
    @click="$emit('click', product.slug)"
  >
    <div class="product-card-media relative flex aspect-[16/10] shrink-0 items-center justify-center overflow-hidden transition-colors duration-500">
      <div v-if="isSoldOut(product)" class="absolute inset-0 z-10 bg-black/45"></div>
      <div
        v-if="isSoldOut(product)"
        class="absolute left-1/2 top-1/2 z-30 -translate-x-1/2 -translate-y-1/2 rounded-md bg-rose-500/90 px-3 py-1.5 text-xs font-bold tracking-widest text-white shadow-lg backdrop-blur-sm"
      >
        {{ t('products.stockStatus.outOfStock') }}
      </div>

      <img
        v-if="product.images && getFirstImageUrl(product.images)"
        :src="getFirstImageUrl(product.images)"
        :alt="getLocalizedText(product.title)"
        loading="lazy"
        class="h-full w-full object-cover transition-all duration-700 ease-out"
        :class="isSoldOut(product) ? 'grayscale brightness-75' : 'group-hover:scale-105'"
      />
      <img
        v-else-if="product.category?.icon"
        :src="getImageUrl(product.category.icon)"
        :alt="getLocalizedText(product.category?.name)"
        loading="lazy"
        class="h-full w-full object-cover transition-all duration-700 ease-out"
        :class="isSoldOut(product) ? 'grayscale brightness-75' : 'group-hover:scale-105'"
      />
      <div v-else class="flex h-full w-full items-center justify-center text-slate-500">
        <svg class="h-8 w-8 md:h-10 md:w-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="1.5"
            d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
          />
        </svg>
      </div>

      <div
        v-if="product.category?.name"
        class="absolute right-3 top-3 z-20 max-w-[70%]"
      >
        <span class="product-card-category truncate px-2.5 py-1 text-[10px] font-semibold">
          {{ getLocalizedText(product.category.name) }}
        </span>
      </div>
    </div>

    <div class="flex flex-1 flex-col gap-2.5 p-4">
      <div class="flex flex-wrap gap-1.5">
        <span
          v-if="hasPromotionPrice(product)"
          class="theme-badge theme-badge-danger text-[10px] font-semibold"
        >
          {{ t('products.promotionTag') }}
        </span>
        <span
          v-else-if="hasPromotionRules(product)"
          class="theme-badge theme-badge-danger text-[10px] font-semibold"
        >
          {{ t('products.promotionBadge') }}
        </span>

        <span
          v-if="product.purchase_type"
          class="theme-badge text-[10px] font-semibold"
          :class="product.purchase_type === 'guest' ? 'theme-badge-warning' : 'theme-badge-success'"
        >
          {{ getPurchaseTypeLabel(product.purchase_type) }}
        </span>

        <span
          v-if="product.tags && product.tags.length > 0"
          class="theme-badge theme-badge-neutral text-[10px] font-semibold"
        >
          {{ product.tags[0] }}
        </span>
      </div>

      <h3
        class="product-card-title min-h-[2.6rem] text-[13px] font-bold leading-relaxed theme-text-primary line-clamp-2"
        :title="getLocalizedText(product.title)"
      >
        {{ getLocalizedText(product.title) }}
      </h3>

      <div class="mt-auto flex items-end justify-between gap-3">
        <div class="min-w-0 flex flex-col">
          <div v-if="hasPromotionPrice(product)" class="mb-0.5 text-[10px] text-slate-400 line-through">
            {{ formatPrice(product.price_amount, siteCurrency) }}
          </div>

          <div
            class="product-card-price flex items-baseline gap-[2px] text-lg font-black tracking-tight"
            :class="isSoldOut(product) ? 'text-gray-500' : 'text-red-500'"
          >
            <span v-if="getPriceObj(getDisplayPriceAmount(product)).prefix" class="text-xs font-bold">
              {{ getPriceObj(getDisplayPriceAmount(product)).prefix }}
            </span>
            <span>{{ getPriceObj(getDisplayPriceAmount(product)).value }}</span>
            <span v-if="getPriceObj(getDisplayPriceAmount(product)).suffix" class="text-xs font-bold">
              {{ getPriceObj(getDisplayPriceAmount(product)).suffix }}
            </span>
          </div>
        </div>

        <button
          type="button"
          class="product-card-cta flex h-8 w-8 shrink-0 items-center justify-center rounded-lg border transition-all"
          :class="isSoldOut(product)
            ? 'cursor-not-allowed border-gray-200 text-gray-400 opacity-40 dark:border-gray-700 dark:text-gray-600'
            : 'border-gray-200 text-gray-500 hover:border-gray-400 hover:bg-gray-100 hover:text-gray-900 dark:border-gray-700 dark:hover:border-gray-500 dark:hover:bg-gray-800 dark:hover:text-white'"
          :disabled="isSoldOut(product)"
          @click.stop="$emit('quickBuy', product)"
        >
          <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="2">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 100 4 2 2 0 000-4z"
            ></path>
          </svg>
        </button>
      </div>

      <div class="product-card-meta flex min-h-[2rem] min-w-0 flex-wrap items-center justify-between gap-2 border-t pt-2.5">
        <div class="flex min-w-0 items-center space-x-1">
          <svg class="h-3 w-3 shrink-0" :class="getStockIconColor(product.stock_status)" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2.5"
              d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4"
            ></path>
          </svg>
          <span class="overflow-hidden text-ellipsis whitespace-nowrap text-[10px] font-semibold theme-text-secondary sm:text-[11px]">
            <span class="hidden sm:inline">{{ t('quickBuy.stock') }}: </span>
            <span class="font-bold" :class="getStockTextColor(product.stock_status)">
              {{ getStockStatusLabel(product) }}
            </span>
          </span>
        </div>

        <div v-if="product.fulfillment_type" class="flex min-w-0 items-center space-x-1">
          <svg
            class="h-3 w-3 shrink-0 text-indigo-500/70"
            :class="isSoldOut(product) ? 'grayscale text-gray-400' : ''"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M13 10V3L4 14h7v7l9-11h-7z"></path>
          </svg>
          <span class="overflow-hidden text-ellipsis whitespace-nowrap text-[10px] font-semibold theme-text-secondary sm:text-[11px]">
            <span class="hidden sm:inline">{{ t('productDetail.fulfillmentLabel') }}: </span>
            <span
              class="font-bold text-indigo-600 dark:text-indigo-400"
              :class="isSoldOut(product) ? 'text-gray-500 dark:text-gray-500' : ''"
            >
              {{ getFulfillmentTypeLabel(product.fulfillment_type) }}
            </span>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { getFirstImageUrl, getImageUrl } from '../utils/image'
import { useLocalized, useProductLabels } from '../composables/useProduct'

withDefaults(defineProps<{
  product: any
  index?: number
  maxTags?: number
  animationStep?: number
}>(), {
  index: 0,
  maxTags: 2,
  animationStep: 50,
})

defineEmits<{
  click: [slug: string]
  quickBuy: [product: any]
}>()

const { t } = useI18n()
const { getLocalizedText, siteCurrency, formatPrice } = useLocalized()
const {
  getPurchaseTypeLabel,
  getFulfillmentTypeLabel,
  getStockStatusLabel,
  isSoldOut,
  hasPromotionPrice,
  getPromotionPriceAmount,
  hasPromotionRules,
} = useProductLabels()

const getDisplayPriceAmount = (item: any) => {
  return hasPromotionPrice(item) ? getPromotionPriceAmount(item) : item?.price_amount
}

const getStockTextColor = (status: string) => {
  if (status === 'out_of_stock') return 'text-gray-500 dark:text-gray-500'
  if (status === 'low_stock') return 'text-amber-600 dark:text-amber-400'
  return 'text-emerald-600 dark:text-emerald-400'
}

const getStockIconColor = (status: string) => {
  if (status === 'out_of_stock') return 'text-gray-400 dark:text-gray-600'
  if (status === 'low_stock') return 'text-amber-500/60'
  return 'text-emerald-500/60'
}

const getPriceObj = (amount: any) => {
  const formatted = String(formatPrice(amount, siteCurrency?.value || siteCurrency))
  const match = formatted.match(/^([^\d.,\s]+)?\s*([\d.,]+)\s*([^\d.,\s]+)?$/)

  return {
    prefix: match?.[1] || '',
    value: match?.[2] || formatted,
    suffix: match?.[3] || '',
  }
}
</script>

<style scoped>
.product-card-shell {
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 97%, white 3%),
      color-mix(in oklab, var(--ui-bg-overlay-strong) 92%, transparent)
    );
  border: 1px solid color-mix(in oklab, var(--ui-border) 84%, white 16%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.55),
    0 18px 36px -30px rgba(15, 23, 42, 0.24);
}

.product-card-shell-active:hover {
  border-color: color-mix(in oklab, var(--ui-border-strong) 72%, white 28%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    0 24px 44px -32px rgba(15, 23, 42, 0.3);
}

.product-card-shell-sold {
  border-color: color-mix(in oklab, var(--ui-danger) 24%, var(--ui-border));
}

.product-card-media {
  background:
    radial-gradient(circle at top left, rgba(255, 255, 255, 0.12), transparent 34%),
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-muted) 78%, #0f172a 22%),
      color-mix(in oklab, var(--ui-bg-soft) 38%, #020617 62%)
    );
}

.product-card-media::after {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.04), transparent 28%),
    linear-gradient(180deg, transparent 45%, rgba(15, 23, 42, 0.16));
}

.product-card-category {
  display: inline-flex;
  align-items: center;
  max-width: 100%;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(15, 23, 42, 0.34);
  color: rgba(255, 255, 255, 0.96);
  backdrop-filter: blur(10px);
  box-shadow: 0 8px 18px -16px rgba(15, 23, 42, 0.45);
}

.product-card-title {
  letter-spacing: -0.018em;
}

.product-card-price {
  color: var(--ui-price-primary);
  font-variant-numeric: tabular-nums;
}

.product-card-price.text-red-500 {
  color: color-mix(in oklab, var(--ui-danger) 84%, #ff6b6b 16%);
}

.product-card-cta {
  border-color: color-mix(in oklab, var(--ui-border) 86%, white 14%) !important;
  background: color-mix(in oklab, var(--ui-bg-soft) 72%, transparent);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.36);
}

.product-card-meta {
  border-top-color: color-mix(in oklab, var(--ui-border) 62%, transparent);
}

:global(.dark .product-card-shell) {
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 98%, black 2%),
      color-mix(in oklab, var(--ui-bg-muted) 95%, black 5%)
    );
  border-color: color-mix(in oklab, var(--ui-border) 88%, white 12%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.05),
    0 22px 44px -32px rgba(0, 0, 0, 0.72);
}

:global(.dark .product-card-shell-active:hover) {
  border-color: color-mix(in oklab, var(--ui-border-strong) 82%, white 18%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.06),
    0 28px 50px -34px rgba(0, 0, 0, 0.82);
}

:global(.dark .product-card-category) {
  border-color: rgba(255, 255, 255, 0.12);
  background: color-mix(in oklab, var(--ui-bg-muted) 94%, black 6%);
  color: rgba(255, 255, 255, 0.94);
  backdrop-filter: none;
}

:global(.dark .product-card-cta) {
  background: color-mix(in oklab, var(--ui-bg-muted) 92%, black 8%);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

@media (max-width: 767px) {
  .product-card-category {
    font-size: 9px;
  }
}
</style>
