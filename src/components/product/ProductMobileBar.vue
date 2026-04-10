<template>
  <Transition
    enter-active-class="transition duration-300 ease-out"
    enter-from-class="translate-y-full opacity-0"
    enter-to-class="translate-y-0 opacity-100"
    leave-active-class="transition duration-200 ease-in"
    leave-from-class="translate-y-0 opacity-100"
    leave-to-class="translate-y-full opacity-0">
    <div v-if="visible"
      class="lg:hidden fixed left-0 right-0 bottom-[calc(3.5rem+env(safe-area-inset-bottom,0px))] z-50 theme-panel-strong backdrop-blur-xl border-t theme-border shadow-2xl">
      <div
        class="grid items-center gap-3 px-4 py-3"
        :class="requiresLogin ? 'grid-cols-[auto_minmax(0,1fr)]' : 'grid-cols-[auto_minmax(0,1fr)_minmax(0,1.15fr)]'"
      >
        <div class="flex shrink-0 items-center overflow-hidden rounded-xl border theme-border bg-white/70 dark:bg-white/[0.03]">
          <button
            type="button"
            class="flex h-11 w-10 items-center justify-center theme-text-secondary transition-colors hover:bg-gray-50 disabled:opacity-30 dark:hover:bg-white/5"
            :disabled="!canDecrease"
            @click="$emit('decrementQuantity')"
          >
            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="2.5">
              <path stroke-linecap="round" d="M20 12H4" />
            </svg>
          </button>
          <span class="flex h-11 min-w-[2.8rem] items-center justify-center border-x theme-border px-3 text-sm font-semibold theme-text-primary tabular-nums">
            {{ quantity }}
          </span>
          <button
            type="button"
            class="flex h-11 w-10 items-center justify-center theme-text-secondary transition-colors hover:bg-gray-50 disabled:opacity-30 dark:hover:bg-white/5"
            :disabled="!canIncrease"
            @click="$emit('incrementQuantity')"
          >
            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="2.5">
              <path stroke-linecap="round" d="M12 4v16m8-8H4" />
            </svg>
          </button>
        </div>

        <!-- Actions -->
        <button v-if="requiresLogin" @click="$emit('goLogin')"
          class="min-w-0 px-5 py-3 theme-btn-primary font-bold rounded-xl text-sm min-h-[44px]">
          {{ t('productDetail.loginToBuy') }}
        </button>
        <template v-else>
          <button @click="$emit('addToCart')" :disabled="!canPurchase"
            class="min-w-0 px-4 py-3 border theme-btn-secondary font-bold rounded-xl text-sm cursor-pointer disabled:cursor-not-allowed disabled:opacity-50 min-h-[44px]">
            {{ t('productDetail.addToCart') }}
          </button>
          <button @click="$emit('buyNow')" :disabled="!canPurchase"
            class="min-w-0 px-5 py-3 theme-btn-primary font-bold rounded-xl text-sm cursor-pointer disabled:cursor-not-allowed disabled:opacity-50 min-h-[44px]">
            {{ t('productDetail.buyNow') }}
          </button>
        </template>
      </div>
    </div>
  </Transition>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

defineProps<{
  visible: boolean
  requiresLogin: boolean
  canPurchase: boolean
  quantity: number
  canDecrease: boolean
  canIncrease: boolean
  showMemberPrice: boolean
  memberPriceDisplay: string
  showSkuPromotionPrice: boolean
  skuPromotionPriceDisplay: string
  showSkuPrice: boolean
  skuPriceDisplay: string
  showProductPromotionPrice: boolean
  productPromotionPriceDisplay: string
  productPriceDisplay: string
}>()

defineEmits<{
  addToCart: []
  buyNow: []
  goLogin: []
  decrementQuantity: []
  incrementQuantity: []
}>()
</script>
