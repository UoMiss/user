<template>
  <div
    class="product-detail-page min-h-screen theme-page pt-24 pb-44 lg:pb-16">
    <div class="container mx-auto px-4">
      <!-- Loading Skeleton -->
      <div v-if="loading" class="space-y-8">
        <div class="h-5 w-48 rounded theme-skeleton"></div>
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-0 theme-panel border rounded-3xl overflow-hidden">
          <div class="p-4 md:p-8 theme-surface-soft border-r theme-border">
            <div class="h-[300px] md:h-[500px] rounded-xl theme-skeleton"></div>
            <div class="mt-4 flex gap-3 overflow-hidden">
              <div v-for="i in 4" :key="i" class="w-16 h-16 rounded-lg theme-skeleton shrink-0"></div>
            </div>
          </div>
          <div class="p-6 md:p-12 space-y-6">
            <div class="h-3 w-24 rounded theme-skeleton"></div>
            <div class="h-10 w-3/4 rounded theme-skeleton"></div>
            <div class="flex gap-2">
              <div class="h-6 w-16 rounded-full theme-skeleton"></div>
              <div class="h-6 w-16 rounded-full theme-skeleton"></div>
              <div class="h-6 w-16 rounded-full theme-skeleton"></div>
            </div>
            <div class="h-14 w-48 rounded theme-skeleton"></div>
            <div class="h-4 w-full rounded theme-skeleton"></div>
            <div class="h-4 w-2/3 rounded theme-skeleton"></div>
            <div class="flex gap-4 mt-8">
              <div class="h-14 flex-1 rounded-xl theme-skeleton"></div>
              <div class="h-14 flex-1 rounded-xl theme-skeleton"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Product Content -->
      <div v-else-if="product">
        <!-- Breadcrumb -->
        <nav class="mb-8 flex items-center space-x-2 text-sm theme-text-muted font-medium">
          <router-link to="/" class="theme-link-muted">{{ t('nav.home')
          }}</router-link>
          <span>/</span>
          <router-link to="/products" class="theme-link-muted">{{
            t('nav.products') }}</router-link>
          <span>/</span>
          <span class="theme-text-primary truncate max-w-[200px]">{{ getLocalizedText(product.title)
          }}</span>
        </nav>

        <!-- Hero + Purchase Layout -->
        <div class="mb-8 space-y-6">
          <section
            class="relative overflow-hidden rounded-[2rem] border border-black/5 bg-[#d7d9df] shadow-[0_26px_70px_-52px_rgba(15,23,42,0.45)] dark:border-white/10 dark:bg-[#202530]"
          >
            <div
              class="pointer-events-none absolute inset-0 bg-[linear-gradient(120deg,transparent_0%,rgba(255,255,255,0.42)_28%,transparent_29%,transparent_44%,rgba(255,255,255,0.22)_45%,transparent_46%,transparent_67%,rgba(255,255,255,0.16)_68%,transparent_69%)] opacity-80 dark:bg-[linear-gradient(120deg,transparent_0%,rgba(255,255,255,0.05)_28%,transparent_29%,transparent_44%,rgba(255,255,255,0.035)_45%,transparent_46%,transparent_67%,rgba(255,255,255,0.025)_68%,transparent_69%)] dark:opacity-100"
            ></div>
            <div class="relative flex flex-col gap-6 px-5 py-5 sm:px-6 sm:py-6 lg:flex-row lg:items-center lg:gap-8 lg:px-8 lg:py-7">
              <div class="shrink-0">
                <div class="rounded-[1.6rem] bg-white/25 p-2 shadow-[0_18px_38px_-28px_rgba(15,23,42,0.45)] backdrop-blur-sm dark:bg-white/[0.05]">
                  <div class="h-28 w-28 overflow-hidden rounded-[1.25rem] border border-white/45 bg-white/80 shadow-sm sm:h-36 sm:w-36 lg:h-40 lg:w-40 dark:border-white/10 dark:bg-slate-900/80">
                    <img
                      v-if="heroDisplayImage"
                      :src="heroDisplayImage"
                      :alt="getLocalizedText(product.title)"
                      class="h-full w-full object-cover"
                    />
                    <div
                      v-else
                      class="flex h-full w-full items-center justify-center bg-slate-100 text-slate-400 dark:bg-slate-900 dark:text-slate-600"
                    >
                      <svg class="h-12 w-12" fill="currentColor" viewBox="0 0 20 20">
                        <path
                          fill-rule="evenodd"
                          d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z"
                          clip-rule="evenodd"
                        />
                      </svg>
                    </div>
                  </div>
                  <div v-if="images.length > 1" class="mt-3 flex gap-2 overflow-x-auto pb-1">
                    <button
                      v-for="(image, index) in images"
                      :key="index"
                      type="button"
                      class="h-10 w-10 shrink-0 overflow-hidden rounded-xl border transition-all"
                      :class="currentImage === image
                        ? 'border-white/80 bg-white shadow-[0_10px_24px_-18px_rgba(15,23,42,0.45)] dark:border-white/35 dark:bg-slate-900'
                        : 'border-transparent bg-white/35 hover:border-white/55 dark:bg-white/[0.04] dark:hover:border-white/20'"
                      @click="currentImage = image"
                    >
                      <img :src="image" :alt="`Image ${index + 1}`" class="h-full w-full object-cover" loading="lazy" />
                    </button>
                  </div>
                </div>
              </div>

              <div class="min-w-0 flex-1">
                <div class="mb-4 flex flex-wrap items-center gap-2">
                  <span
                    v-if="categoryName"
                    class="inline-flex items-center rounded-full border border-white/50 bg-white/70 px-3 py-1 text-xs font-medium text-slate-700 shadow-sm backdrop-blur-sm dark:border-white/12 dark:bg-white/[0.05] dark:text-slate-200"
                  >
                    {{ categoryName }}
                  </span>
                  <span
                    class="inline-flex items-center rounded-full border border-white/45 bg-white/70 px-3 py-1 text-xs font-medium text-slate-700 shadow-sm backdrop-blur-sm dark:border-white/12 dark:bg-white/[0.05] dark:text-slate-200"
                  >
                    {{ getPurchaseTypeLabel(product.purchase_type) }}
                  </span>
                  <span
                    class="inline-flex items-center rounded-full border border-white/45 bg-white/70 px-3 py-1 text-xs font-medium text-slate-700 shadow-sm backdrop-blur-sm dark:border-white/12 dark:bg-white/[0.05] dark:text-slate-200"
                  >
                    {{ getFulfillmentTypeLabel(product.fulfillment_type) }}
                  </span>
                </div>

                <h1 class="mb-4 text-[1.9rem] font-semibold leading-[1.08] tracking-[-0.032em] text-white sm:text-[2.3rem] lg:text-[2.7rem]">
                  {{ getLocalizedText(product.title) }}
                </h1>

                <div v-if="product.tags && product.tags.length > 0" class="mb-4 flex flex-wrap gap-2">
                  <span
                    v-for="(tag, index) in product.tags"
                    :key="index"
                    class="inline-flex items-center rounded-full border border-white/35 bg-white/18 px-3 py-1 text-xs font-medium text-white/88 backdrop-blur-sm"
                  >
                    {{ tag }}
                  </span>
                </div>

                <p v-if="getLocalizedText(product.description)" class="max-w-3xl text-sm leading-6 text-white/82 sm:text-base">
                  {{ getLocalizedText(product.description) }}
                </p>
              </div>
            </div>
          </section>

          <section class="grid grid-cols-1 gap-6 xl:grid-cols-[minmax(0,1.72fr)_minmax(340px,0.9fr)] xl:gap-7">
            <div
              class="rounded-[2rem] border border-black/5 bg-white/95 p-5 shadow-[0_20px_55px_-40px_rgba(15,23,42,0.32)] dark:border-white/10 dark:bg-slate-950/72 sm:p-6"
            >
              <div class="mb-5 flex flex-wrap items-center justify-between gap-3 border-b border-slate-200/80 pb-4 dark:border-white/10">
                <div>
                  <p class="text-sm font-medium text-slate-500 dark:text-slate-400">{{ t('productDetail.skuTitle') }}</p>
                  <h2 class="mt-1 text-xl font-semibold text-slate-900 dark:text-white">
                    {{ t('productDetail.skuTitle') }}
                  </h2>
                </div>
                <span
                  v-if="activeSkus.length"
                  class="inline-flex items-center rounded-full border border-slate-200 bg-slate-50 px-3 py-1 text-xs font-medium text-slate-500 dark:border-white/10 dark:bg-white/[0.04] dark:text-slate-300"
                >
                  {{ activeSkus.length }} SKU
                </span>
              </div>

              <div v-if="activeSkus.length" class="grid grid-cols-1 gap-3 md:grid-cols-2">
                <button
                  v-for="sku in activeSkus"
                  :key="sku.id"
                  type="button"
                  class="group relative flex min-h-[106px] items-center justify-between rounded-[1.1rem] border px-4 py-3.5 text-left transition-all"
                  :class="[
                    normalizeSkuId(sku.id) === selectedSkuId
                      ? 'border-orange-400 bg-white shadow-[0_18px_36px_-34px_rgba(249,115,22,0.55)] ring-1 ring-orange-200/70 dark:border-orange-400/70 dark:bg-slate-950 dark:ring-orange-400/15'
                      : 'border-slate-200 bg-white shadow-[0_12px_26px_-28px_rgba(15,23,42,0.16)] hover:-translate-y-0.5 hover:border-slate-300 hover:shadow-[0_18px_34px_-30px_rgba(15,23,42,0.2)] dark:border-white/10 dark:bg-white/[0.02] dark:hover:border-white/16',
                    isSkuPurchasable(sku) ? '' : 'cursor-not-allowed opacity-55 border-dashed',
                  ]"
                  :disabled="!isSkuPurchasable(sku)"
                  @click="selectedSkuId = normalizeSkuId(sku.id)"
                >
                  <span
                    v-if="normalizeSkuId(sku.id) === selectedSkuId"
                    class="absolute right-3 top-3 h-2 w-2 rounded-full bg-orange-500 shadow-[0_0_0_4px_rgba(249,115,22,0.14)] dark:bg-orange-300 dark:shadow-[0_0_0_4px_rgba(251,146,60,0.12)]"
                  ></span>
                  <div class="flex min-w-0 items-center gap-3.5">
                    <div class="h-14 w-14 shrink-0 overflow-hidden rounded-[0.95rem] border border-black/5 bg-slate-100 dark:border-white/10 dark:bg-slate-900/90">
                      <img
                        v-if="heroDisplayImage"
                        :src="heroDisplayImage"
                        :alt="skuDisplayText(sku)"
                        class="h-full w-full object-cover"
                      />
                      <div
                        v-else
                        class="flex h-full w-full items-center justify-center text-slate-400 dark:text-slate-600"
                      >
                        <svg class="h-6 w-6" fill="currentColor" viewBox="0 0 20 20">
                          <path
                            fill-rule="evenodd"
                            d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z"
                            clip-rule="evenodd"
                          />
                        </svg>
                      </div>
                    </div>
                    <div class="min-w-0 flex-1">
                      <div class="flex items-start gap-2">
                        <p class="line-clamp-2 text-[0.96rem] font-semibold leading-6 text-slate-900 dark:text-white">
                          {{ skuDisplayText(sku) }}
                        </p>
                        <span
                          v-if="isSkuPurchasable(sku) && product.fulfillment_type === 'auto'"
                          class="mt-1 inline-flex h-5 w-5 shrink-0 items-center justify-center rounded-full bg-emerald-500/12 text-emerald-600 dark:bg-emerald-400/10 dark:text-emerald-300"
                        >
                          <svg class="h-3 w-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2" d="M13 2L3 14h7l-1 8 10-12h-7l1-8z" />
                          </svg>
                        </span>
                      </div>
                      <span
                        class="mt-2 inline-flex rounded-full border px-2.5 py-1 text-[11px] font-medium"
                        :class="skuStockBadgeClass(sku)"
                      >
                        {{ skuStockText(sku) }}
                      </span>
                    </div>
                  </div>

                  <div class="ml-4 flex shrink-0 flex-col items-end justify-center text-right">
                    <span
                      v-if="getSkuCardOriginalPrice(sku) !== null"
                      class="text-xs font-medium text-slate-400 line-through dark:text-slate-500"
                    >
                      {{ formatPrice(getSkuCardOriginalPrice(sku)!, siteCurrency) }}
                    </span>
                    <span class="mt-1 text-[1.08rem] font-semibold text-orange-600 dark:text-orange-300">
                      {{ formatPrice(getSkuCardCurrentPrice(sku), siteCurrency) }}
                    </span>
                  </div>
                </button>
              </div>

              <p v-if="requiresSKUSelection" class="mt-3 text-xs font-medium text-amber-500">
                {{ t('productDetail.skuRequired') }}
              </p>

              <div
                v-if="hasPromotionRules(product)"
                class="mt-6 rounded-[1.4rem] border border-amber-200/70 bg-amber-50/80 px-4 py-4 shadow-[0_16px_40px_-36px_rgba(217,119,6,0.55)] dark:border-amber-400/20 dark:bg-amber-500/10"
              >
                <h3 class="mb-3 flex items-center gap-2 text-sm font-semibold text-amber-700 dark:text-amber-300">
                  <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A2 2 0 013 12V7a4 4 0 014-4z" />
                  </svg>
                  {{ t('products.promotionRulesTitle') }}
                </h3>
                <ul class="space-y-2">
                  <li
                    v-for="rule in getPromotionRules(product)"
                    :key="rule.id"
                    class="flex items-start gap-2 text-sm text-amber-700 dark:text-amber-200/90"
                  >
                    <span class="mt-1 h-1.5 w-1.5 shrink-0 rounded-full bg-amber-400 dark:bg-amber-300"></span>
                    <span>{{ formatPromotionRule(rule) }}</span>
                  </li>
                </ul>
              </div>
            </div>

            <div class="space-y-5 xl:sticky xl:top-28 xl:self-start">
              <div
                class="rounded-[1.8rem] border border-black/5 bg-white/95 px-5 py-5 shadow-[0_18px_45px_-38px_rgba(15,23,42,0.3)] dark:border-white/10 dark:bg-slate-950/75 sm:px-6"
              >
                <div class="flex items-center justify-between gap-4">
                  <div>
                    <p class="text-sm font-medium text-slate-500 dark:text-slate-400">{{ t('quickBuy.quantity') }}</p>
                    <h2 class="mt-1 text-lg font-semibold text-slate-900 dark:text-white">
                      {{ t('productDetail.quantity') }}
                    </h2>
                  </div>
                  <div class="flex items-center overflow-hidden rounded-2xl border border-slate-200 bg-white shadow-sm dark:border-white/10 dark:bg-white/[0.04]">
                    <button
                      type="button"
                      class="flex h-12 w-12 items-center justify-center text-slate-500 transition-colors hover:bg-slate-50 hover:text-slate-800 disabled:opacity-30 dark:text-slate-300 dark:hover:bg-white/[0.06] dark:hover:text-white"
                      :disabled="quantity <= 1"
                      @click="quantity = Math.max(1, quantity - 1)"
                    >
                      <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="2.5">
                        <path stroke-linecap="round" d="M20 12H4" />
                      </svg>
                    </button>
                    <input
                      type="text"
                      inputmode="numeric"
                      class="h-12 w-16 border-x border-slate-200 bg-transparent text-center text-base font-semibold tabular-nums text-slate-900 outline-none [appearance:textfield] dark:border-white/10 dark:text-white [&::-webkit-inner-spin-button]:appearance-none [&::-webkit-outer-spin-button]:appearance-none"
                      :value="quantity"
                      @change="handleQuantityInput($event)"
                      @keydown.enter.prevent="($event.target as HTMLInputElement)?.blur()"
                    />
                    <button
                      type="button"
                      class="flex h-12 w-12 items-center justify-center text-slate-500 transition-colors hover:bg-slate-50 hover:text-slate-800 disabled:opacity-30 dark:text-slate-300 dark:hover:bg-white/[0.06] dark:hover:text-white"
                      :disabled="quantityEffectiveLimit !== null && quantity >= quantityEffectiveLimit"
                      @click="quantity = quantity + 1"
                    >
                      <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="2.5">
                        <path stroke-linecap="round" d="M12 4v16m8-8H4" />
                      </svg>
                    </button>
                  </div>
                </div>
              </div>

              <div
                class="rounded-[1.9rem] border border-black/5 bg-white/95 px-5 py-5 shadow-[0_22px_58px_-40px_rgba(15,23,42,0.34)] dark:border-white/10 dark:bg-slate-950/78 sm:px-6 sm:py-6"
              >
                <div class="border-b border-slate-200 pb-5 dark:border-white/10">
                  <p class="text-sm font-medium text-slate-500 dark:text-slate-400">{{ t('cart.summaryTitle') }}</p>
                  <div class="mt-3 flex items-start justify-between gap-4">
                    <div class="min-w-0">
                      <p class="text-[2rem] font-semibold tracking-[-0.035em] text-orange-600 dark:text-orange-300 sm:text-[2.4rem]">
                        {{ summaryTotalPriceDisplay }}
                      </p>
                      <p
                        v-if="summaryOriginalTotalPriceDisplay"
                        class="mt-1 text-sm font-medium text-slate-400 line-through dark:text-slate-500"
                      >
                        {{ summaryOriginalTotalPriceDisplay }}
                      </p>
                    </div>
                    <span
                      v-if="summaryPriceBadge"
                      class="inline-flex shrink-0 rounded-full border border-orange-200 bg-orange-50 px-3 py-1 text-xs font-semibold text-orange-600 dark:border-orange-400/20 dark:bg-orange-500/10 dark:text-orange-300"
                    >
                      {{ summaryPriceBadge }}
                    </span>
                  </div>
                  <div
                    v-if="summaryPriceNote"
                    class="mt-4 rounded-2xl border border-orange-100 bg-orange-50/80 px-3.5 py-3 text-sm font-medium text-orange-700 dark:border-orange-400/15 dark:bg-orange-500/10 dark:text-orange-200/90"
                  >
                    {{ summaryPriceNote }}
                  </div>
                </div>

                <div
                  v-if="selectedSku"
                  class="mt-5 rounded-[1.35rem] border border-black/5 bg-slate-50/85 px-4 py-3.5 dark:border-white/10 dark:bg-white/[0.04]"
                >
                  <p class="text-xs font-medium uppercase tracking-[0.18em] text-slate-400 dark:text-slate-500">{{ t('cart.skuLabel') }}</p>
                  <div class="mt-2 flex items-start justify-between gap-4">
                    <p class="line-clamp-2 text-sm font-semibold leading-6 text-slate-900 dark:text-white">
                      {{ skuDisplayText(selectedSku) }}
                    </p>
                    <span class="shrink-0 text-sm font-semibold text-rose-600 dark:text-rose-300">
                      {{ formatPrice(getSkuCardCurrentPrice(selectedSku), siteCurrency) }}
                    </span>
                  </div>
                </div>

                <div class="mt-5 rounded-[1.35rem] border border-black/5 bg-slate-50/85 px-4 py-4 dark:border-white/10 dark:bg-white/[0.04]">
                  <div class="flex items-center justify-between gap-4 text-sm">
                    <span class="text-slate-500 dark:text-slate-400">{{ t('quickBuy.stock') }}</span>
                    <span class="font-medium text-slate-900 dark:text-white">{{ getStockStatusLabel(product) }}</span>
                  </div>
                  <div class="mt-3 flex items-center justify-between gap-4 text-sm border-t border-slate-200 pt-3 dark:border-white/10">
                    <span class="text-slate-500 dark:text-slate-400">{{ t('products.fulfillmentTypeLabel') }}</span>
                    <span class="font-medium text-slate-900 dark:text-white">{{ getFulfillmentTypeLabel(product.fulfillment_type) }}</span>
                  </div>
                  <div class="mt-3 flex items-center justify-between gap-4 text-sm border-t border-slate-200 pt-3 dark:border-white/10">
                    <span class="text-slate-500 dark:text-slate-400">{{ t('productPurchase.label') }}</span>
                    <span class="font-medium text-slate-900 dark:text-white">{{ getPurchaseTypeLabel(product.purchase_type) }}</span>
                  </div>
                </div>

                <p v-if="cannotPurchaseReason" class="mt-5 rounded-2xl border theme-alert-danger px-4 py-3 text-sm font-semibold">
                  {{ cannotPurchaseReason }}
                </p>
                <p v-if="purchaseWarning" class="mt-4 rounded-2xl border theme-alert-warning px-4 py-3 text-sm font-semibold">
                  {{ purchaseWarning }}
                </p>

                <div class="mt-6 hidden space-y-3 lg:block">
                  <button
                    v-if="requiresLogin"
                    @click="goLogin"
                    class="w-full rounded-[1.15rem] theme-btn-primary px-6 py-4 font-bold transition-colors min-h-[48px]"
                  >
                    {{ t('productDetail.loginToBuy') }}
                  </button>
                  <div v-else class="grid grid-cols-[minmax(0,0.82fr)_minmax(0,1.18fr)] gap-3">
                    <button
                      @click="addToCart"
                      :disabled="!canPurchase"
                      class="rounded-[1.15rem] bg-[#f7b324] px-5 py-4 font-bold text-slate-900 shadow-[0_16px_36px_-30px_rgba(247,179,36,0.95)] transition-colors hover:bg-[#f2aa13] disabled:cursor-not-allowed disabled:opacity-50 min-h-[48px]"
                    >
                      {{ t('productDetail.addToCart') }}
                    </button>
                    <button
                      @click="buyNow"
                      :disabled="!canPurchase"
                      class="rounded-[1.15rem] bg-[linear-gradient(135deg,#ff6a1a,#ff4a0d)] px-5 py-4 font-bold text-white shadow-[0_20px_44px_-30px_rgba(255,90,26,0.9)] transition-opacity hover:opacity-95 disabled:cursor-not-allowed disabled:opacity-50 min-h-[48px]"
                    >
                      {{ t('productDetail.buyNow') }}
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </section>
        </div>

        <!-- Details Content Card -->
        <div v-if="product.content"
          class="theme-panel backdrop-blur-xl border rounded-3xl overflow-hidden mb-12 p-6 md:p-8 lg:p-12 relative">
          <h2
            class="text-2xl font-bold mb-8 theme-text-primary flex items-center gap-3 border-b theme-border pb-6">
            <span class="w-1.5 h-8 theme-accent-stick rounded-full"></span>
            {{ t('productDetail.details') }}
          </h2>
          <div v-html="processHtmlForDisplay(getLocalizedText(product.content))"
            class="prose prose-gray dark:prose-invert prose-lg max-w-none theme-prose">
          </div>
        </div>

        <!-- Back Button -->
        <div class="mb-12 text-center">
          <router-link to="/products"
            class="inline-flex items-center space-x-2 theme-link-muted transition-colors border-b border-transparent hover:border-current pb-1">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
            </svg>
            <span>{{ t('productDetail.backToProducts') }}</span>
          </router-link>
        </div>

        <!-- Mobile Fixed Purchase Bar -->
        <ProductMobileBar
          :visible="!!product && !loading"
          :requires-login="requiresLogin"
          :can-purchase="canPurchase"
          :show-member-price="mobileBarShowMemberPrice"
          :member-price-display="mobileBarMemberPriceDisplay"
          :show-sku-promotion-price="mobileBarShowSkuPromotionPrice"
          :sku-promotion-price-display="mobileBarSkuPromotionPriceDisplay"
          :show-sku-price="mobileBarShowSkuPrice"
          :sku-price-display="mobileBarSkuPriceDisplay"
          :show-product-promotion-price="mobileBarShowProductPromotionPrice"
          :product-promotion-price-display="mobileBarProductPromotionPriceDisplay"
          :product-price-display="mobileBarProductPriceDisplay"
          @add-to-cart="addToCart"
          @buy-now="buyNow"
          @go-login="goLogin"
        />
      </div>

      <!-- Error State -->
      <div v-else
        class="text-center py-24 theme-panel rounded-3xl border backdrop-blur-sm theme-slide-up">
        <svg class="w-20 h-20 mx-auto theme-text-muted mb-6" fill="none" stroke="currentColor"
          viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <p class="theme-text-muted text-xl mb-8">
          {{ t('productDetail.notFound') }}
        </p>
        <div class="flex flex-wrap items-center justify-center gap-4">
          <button
            @click="loadProduct"
            class="inline-flex items-center gap-2 theme-btn-primary px-8 py-3 rounded-full font-bold hover:scale-105 transition-transform"
          >
            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
            </svg>
            {{ t('errorBoundary.retry') }}
          </button>
          <router-link to="/products"
            class="inline-block border theme-btn-secondary px-8 py-3 rounded-full font-bold hover:scale-105 transition-transform">
            {{ t('productDetail.backToProducts') }}
          </router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { useAppStore } from '../stores/app'
import { productAPI } from '../api'
import { getImageUrl } from '../utils/image'
import { processHtmlForDisplay } from '../utils/content'
import { useCartStore } from '../stores/cart'
import { useBuyNowStore } from '../stores/buyNow'
import { useUserAuthStore } from '../stores/userAuth'
import { useHead } from '@unhead/vue'
// centsToAmount used internally by composable
import { buildSkuDisplayText, normalizeSkuId } from '../utils/sku'
import { useLocalized, useProductLabels } from '../composables/useProduct'
import { toast } from '../composables/useToast'
import ProductMobileBar from '../components/product/ProductMobileBar.vue'

const route = useRoute()
const router = useRouter()
const { t } = useI18n()
const appStore = useAppStore()
const cartStore = useCartStore()
const buyNowStore = useBuyNowStore()
const userAuthStore = useUserAuthStore()

const { getLocalizedText, siteCurrency, formatPrice } = useLocalized()
const { getPurchaseTypeLabel, getFulfillmentTypeLabel, getStockStatusLabel, hasPromotionPrice, getPromotionPriceAmount, getPromotionSaveAmount, hasSkuPromotionPrice, getSkuPromotionPriceAmount, getSkuPromotionSaveAmount, hasPromotionRules, getPromotionRules } = useProductLabels()

const formatPromotionRule = (rule: any) => {
  const amount = formatPrice(rule.min_amount, siteCurrency.value)
  const value = rule.type === 'percent' ? String(rule.value) : formatPrice(rule.value, siteCurrency.value)
  const hasMin = Number(rule.min_amount) > 0
  switch (rule.type) {
    case 'percent':
      return hasMin ? t('products.promotionHintPercent', { amount, value }) : t('products.promotionHintPercentNoMin', { value })
    case 'fixed':
      return hasMin ? t('products.promotionHintFixed', { amount, value }) : t('products.promotionHintFixedNoMin', { value })
    case 'special_price':
      return hasMin ? t('products.promotionHintSpecial', { amount, value }) : t('products.promotionHintSpecialNoMin', { value })
    default:
      return rule.name || ''
  }
}

const loading = ref(true)
const product = ref<any>(null)
const currentImage = ref<string>('')
const selectedSkuId = ref(0)
const quantity = ref(1)
const purchaseWarning = ref('')

const activeSkus = computed(() => {
  const rows = Array.isArray(product.value?.skus) ? product.value.skus : []
  return rows.filter((sku: any) => Boolean(sku?.is_active))
})

const selectedSku = computed(() => {
  if (selectedSkuId.value <= 0) return null
  return activeSkus.value.find((sku: any) => normalizeSkuId(sku?.id) === selectedSkuId.value) || null
})

// 会员价相关
const userMemberLevelId = computed(() => {
  return Number(userAuthStore.user?.member_level_id || 0)
})

const getMemberPriceForSku = (skuId: number): number | null => {
  if (!product.value?.member_prices || !userMemberLevelId.value) return null
  const prices = product.value.member_prices as Array<{ member_level_id: number; sku_id: number; price_amount: number | string }>
  // SKU-level override
  const skuPrice = prices.find((p) => p.member_level_id === userMemberLevelId.value && p.sku_id === skuId)
  if (skuPrice) return Number(skuPrice.price_amount)
  // Product-level override (sku_id = 0)
  const productPrice = prices.find((p) => p.member_level_id === userMemberLevelId.value && p.sku_id === 0)
  if (productPrice) return Number(productPrice.price_amount)
  return null
}

const selectedSkuMemberPrice = computed(() => {
  if (!selectedSku.value) return null
  const skuId = normalizeSkuId(selectedSku.value.id)
  return getMemberPriceForSku(skuId)
})

const hasMemberPrice = computed(() => {
  if (!selectedSkuMemberPrice.value) return false
  const basePrice = Number(selectedSku.value?.price_amount || 0)
  return selectedSkuMemberPrice.value < basePrice
})

const getSkuCardCurrentPrice = (sku: any) => {
  const basePrice = Number(sku?.price_amount || 0)
  const memberPrice = getMemberPriceForSku(normalizeSkuId(sku?.id))
  const promoPrice = hasSkuPromotionPrice(sku) ? Number(getSkuPromotionPriceAmount(sku)) : basePrice
  if (memberPrice !== null && memberPrice < promoPrice) return memberPrice
  return promoPrice
}

const getSkuCardOriginalPrice = (sku: any) => {
  const basePrice = Number(sku?.price_amount || 0)
  const currentPrice = getSkuCardCurrentPrice(sku)
  return currentPrice < basePrice ? basePrice : null
}

const normalizeStockNumber = (value: unknown) => {
  const numberValue = Number(value)
  if (!Number.isFinite(numberValue)) return 0
  return Math.max(Math.floor(numberValue), 0)
}

const normalizeManualStockTotal = (value: unknown) => {
  const numberValue = Number(value)
  if (!Number.isFinite(numberValue)) return 0
  const integerValue = Math.floor(numberValue)
  if (integerValue === -1) return -1
  return Math.max(integerValue, 0)
}

const normalizeOptionalLimitNumber = (value: unknown) => {
  const numberValue = Number(value)
  if (!Number.isFinite(numberValue)) return null
  const integerValue = Math.floor(numberValue)
  if (integerValue <= 0) return null
  return integerValue
}

const shouldEnforceSkuStock = (sku: any) => {
  if (!sku) return false
  if (product.value?.fulfillment_type === 'auto') return true
  if (product.value?.fulfillment_type === 'upstream') return true
  if (product.value?.fulfillment_type !== 'manual') return false
  const total = normalizeManualStockTotal(sku?.manual_stock_total)
  if (total === -1) return false
  return true
}

const skuAvailableStock = (sku: any) => {
  if (!shouldEnforceSkuStock(sku)) return null
  if (product.value?.fulfillment_type === 'upstream') {
    const upstreamStock = Number(sku?.upstream_stock ?? 0)
    if (upstreamStock === -1) return null
    return Math.max(upstreamStock, 0)
  }
  if (product.value?.fulfillment_type === 'auto') {
    const autoStock = Number(sku?.auto_stock_available ?? 0)
    if (autoStock < 0) return null
    return normalizeStockNumber(autoStock)
  }
  const total = normalizeManualStockTotal(sku?.manual_stock_total)
  if (total === -1) return null
  return total
}

const isSkuPurchasable = (sku: any) => {
  const available = skuAvailableStock(sku)
  if (available === null) return true
  return available > 0
}

const skuStockText = (sku: any) => {
  const available = skuAvailableStock(sku)
  if (available === null) return t('productDetail.skuStockUnlimited')
  if (available <= 0) return t('productDetail.skuStockOut')
  return t('productDetail.skuStockRemaining', { count: available })
}

const skuStockBadgeClass = (sku: any) => {
  const available = skuAvailableStock(sku)
  if (available === null) return 'border-slate-200 text-slate-600 dark:border-slate-700 dark:text-slate-300'
  if (available <= 0) return 'border-rose-200 bg-rose-50 text-rose-700 dark:border-rose-700 dark:bg-rose-950/30 dark:text-rose-300'
  if (available <= 5) return 'border-amber-200 bg-amber-50 text-amber-700 dark:border-amber-700 dark:bg-amber-950/30 dark:text-amber-300'
  return 'border-emerald-200 bg-emerald-50 text-emerald-700 dark:border-emerald-700 dark:bg-emerald-950/30 dark:text-emerald-300'
}

const quantityEffectiveLimit = computed(() => {
  const sku = selectedSku.value
  const available = skuAvailableStock(sku)
  const productLimit = normalizeOptionalLimitNumber(product.value?.max_purchase_quantity)
  let limit: number | null = productLimit
  if (available !== null) {
    limit = limit === null ? available : Math.min(limit, available)
  }
  return limit
})

const handleQuantityInput = (event: Event) => {
  const val = parseInt((event.target as HTMLInputElement).value, 10)
  if (isNaN(val) || val < 1) {
    quantity.value = 1
  } else if (quantityEffectiveLimit.value !== null && val > quantityEffectiveLimit.value) {
    quantity.value = quantityEffectiveLimit.value
  } else {
    quantity.value = val
  }
}

const purchaseType = computed(() => product.value?.purchase_type || 'member')
const requiresLogin = computed(() => purchaseType.value === 'member' && !userAuthStore.isAuthenticated)
const requiresSKUSelection = computed(() => activeSkus.value.length > 1 && !selectedSku.value)
const canPurchase = computed(() => {
  if (!product.value) return false
  if (activeSkus.value.length === 0) return false
  if (product.value.is_sold_out) return false
  if (requiresSKUSelection.value) return false
  if (product.value.stock_status === 'out_of_stock') return false
  if (selectedSku.value && !isSkuPurchasable(selectedSku.value)) return false
  return true
})
const cannotPurchaseReason = computed(() => {
  if (!product.value) return ''
  if (requiresLogin.value) return ''
  if (requiresSKUSelection.value) return t('productDetail.skuRequired')
  if (canPurchase.value) return ''
  return t('productDetail.stockUnavailable')
})
const categoryName = computed(() => {
  const category = product.value?.category?.name
  return category ? getLocalizedText(category) : ''
})

const images = computed(() => {
  if (!product.value?.images) return []
  let imageArray: string[] = []
  if (Array.isArray(product.value.images)) {
    imageArray = product.value.images
  } else if (product.value.images.images && Array.isArray(product.value.images.images)) {
    imageArray = product.value.images.images
  }
  return imageArray.map(img => getImageUrl(img))
})

const heroDisplayImage = computed(() => {
  return currentImage.value || images.value[0] || ''
})

const currentUnitPrice = computed(() => {
  if (selectedSku.value) {
    if (hasMemberPrice.value && selectedSkuMemberPrice.value !== null) {
      const skuPromoPrice = hasSkuPromotionPrice(selectedSku.value)
        ? Number(getSkuPromotionPriceAmount(selectedSku.value))
        : Number(selectedSku.value.price_amount || 0)
      if (selectedSkuMemberPrice.value < skuPromoPrice) return selectedSkuMemberPrice.value
    }
    if (hasSkuPromotionPrice(selectedSku.value)) return Number(getSkuPromotionPriceAmount(selectedSku.value))
    return Number(selectedSku.value.price_amount || 0)
  }
  if (product.value && hasPromotionPrice(product.value)) {
    return Number(getPromotionPriceAmount(product.value))
  }
  return Number(product.value?.price_amount || 0)
})

const currentOriginalUnitPrice = computed(() => {
  if (selectedSku.value) {
    const basePrice = Number(selectedSku.value.price_amount || 0)
    return currentUnitPrice.value < basePrice ? basePrice : null
  }
  const basePrice = Number(product.value?.price_amount || 0)
  return currentUnitPrice.value < basePrice ? basePrice : null
})

const summaryTotalPriceDisplay = computed(() => {
  return formatPrice(currentUnitPrice.value * quantity.value, siteCurrency.value)
})

const summaryOriginalTotalPriceDisplay = computed(() => {
  if (currentOriginalUnitPrice.value === null) return ''
  return formatPrice(currentOriginalUnitPrice.value * quantity.value, siteCurrency.value)
})

const summaryPriceBadge = computed(() => {
  if (selectedSku.value && hasMemberPrice.value) return t('products.memberPriceTag')
  if ((selectedSku.value && hasSkuPromotionPrice(selectedSku.value)) || (!selectedSku.value && product.value && hasPromotionPrice(product.value))) {
    return t('products.promotionTag')
  }
  return ''
})

const summaryPriceNote = computed(() => {
  if (selectedSku.value) {
    if (hasMemberPrice.value && selectedSkuMemberPrice.value !== null) {
      return `${t('products.memberPriceTag')} · ${t('products.saveAmount')} ${formatPrice(Number(selectedSku.value.price_amount || 0) - selectedSkuMemberPrice.value, siteCurrency.value)}`
    }
    if (hasSkuPromotionPrice(selectedSku.value)) {
      return `${t('products.saveAmount')} ${formatPrice(getSkuPromotionSaveAmount(selectedSku.value), siteCurrency.value)}`
    }
    return ''
  }
  if (product.value && hasPromotionPrice(product.value)) {
    return `${t('products.saveAmount')} ${formatPrice(getPromotionSaveAmount(product.value), siteCurrency.value)}`
  }
  return ''
})

const skuDisplayText = (sku: any) => {
  return buildSkuDisplayText({
    skuCode: sku?.sku_code,
    specValues: sku?.spec_values,
    fallback: t('productDetail.skuFallback'),
    locale: appStore.locale,
  })
}

const syncSelectedSku = () => {
  const rows = activeSkus.value
  if (rows.length === 0) {
    selectedSkuId.value = 0
    return
  }
  if (rows.length === 1) {
    selectedSkuId.value = normalizeSkuId(rows[0]?.id)
    return
  }
  if (rows.some((sku: any) => normalizeSkuId(sku?.id) === selectedSkuId.value)) {
    return
  }
  const firstAvailable = rows.find((sku: any) => isSkuPurchasable(sku))
  if (firstAvailable) {
    selectedSkuId.value = normalizeSkuId(firstAvailable?.id)
    return
  }
  selectedSkuId.value = normalizeSkuId(rows[0]?.id)
}

const selectedCartQuantity = () => {
  if (!product.value || !selectedSku.value) return 0
  const productId = Number(product.value.id || 0)
  const skuId = normalizeSkuId(selectedSku.value?.id)
  if (productId <= 0 || skuId <= 0) return 0
  const matched = cartStore.items.find((item) => item.productId === productId && normalizeSkuId(item.skuId) === skuId)
  return Number(matched?.quantity || 0)
}

const addToCart = () => {
  if (!product.value) return
  if (!canPurchase.value) return
  purchaseWarning.value = ''
  if (requiresLogin.value) {
    router.push(`/auth/login?redirect=${encodeURIComponent(route.fullPath)}`)
    return
  }
  const sku = selectedSku.value
  const available = skuAvailableStock(sku)
  const cartQty = selectedCartQuantity()
  const nextQuantity = cartQty + quantity.value
  const productLimit = normalizeOptionalLimitNumber(product.value?.max_purchase_quantity)
  let effectiveLimit: number | null = productLimit
  if (available !== null) {
    effectiveLimit = effectiveLimit === null ? available : Math.min(effectiveLimit, available)
  }
  if (effectiveLimit !== null && nextQuantity > effectiveLimit) {
    if (available !== null && effectiveLimit === available && (productLimit === null || available <= productLimit)) {
      purchaseWarning.value = available > 0
        ? (cartQty > 0
            ? t('productDetail.addCartStockExceededWithCart', { count: available, cartCount: cartQty })
            : t('productDetail.addCartStockExceeded', { count: available }))
        : t('productDetail.stockUnavailable')
      return
    }
    purchaseWarning.value = cartQty > 0
      ? t('productDetail.addCartLimitExceededWithCart', { count: effectiveLimit, cartCount: cartQty })
      : t('productDetail.addCartLimitExceeded', { count: effectiveLimit })
    return
  }
  cartStore.addItem({
    productId: product.value.id,
    skuId: normalizeSkuId(sku?.id),
    skuCode: String(sku?.sku_code || ''),
    skuSpecValues: (sku?.spec_values && typeof sku.spec_values === 'object') ? sku.spec_values : undefined,
    skuManualStockTotal: normalizeManualStockTotal(sku?.manual_stock_total),
    skuManualStockLocked: normalizeStockNumber(sku?.manual_stock_locked),
    skuManualStockSold: normalizeStockNumber(sku?.manual_stock_sold),
    skuAutoStockAvailable: normalizeStockNumber(sku?.auto_stock_available),
    skuUpstreamStock: normalizeManualStockTotal(sku?.upstream_stock),
    skuStockEnforced: shouldEnforceSkuStock(sku),
    slug: product.value.slug,
    title: product.value.title,
    priceAmount: String(sku?.price_amount || product.value.price_amount || '0.00'),
    image: images.value[0],
    maxPurchaseQuantity: normalizeOptionalLimitNumber(product.value.max_purchase_quantity) ?? undefined,
    purchaseType: product.value.purchase_type,
    fulfillmentType: product.value.fulfillment_type,
    manualFormSchema: product.value.manual_form_schema || {},
    paymentChannelIds: Array.isArray(product.value.payment_channel_ids) && product.value.payment_channel_ids.length > 0 ? product.value.payment_channel_ids : undefined,
    quantity: quantity.value,
  }, quantity.value)
  toast.success(t('toast.addedToCart'))
}

const buyNow = () => {
  purchaseWarning.value = ''
  if (!canPurchase.value) return
  if (!product.value) return
  if (requiresLogin.value) {
    router.push(`/auth/login?redirect=${encodeURIComponent(route.fullPath)}`)
    return
  }

  const sku = selectedSku.value
  const available = skuAvailableStock(sku)
  const productLimit = normalizeOptionalLimitNumber(product.value?.max_purchase_quantity)
  let limit: number | null = productLimit
  if (available !== null) {
    limit = limit === null ? available : Math.min(limit, available)
  }
  if (limit !== null && quantity.value > limit) {
    purchaseWarning.value = available !== null && limit === available
      ? (available > 0 ? t('productDetail.addCartStockExceeded', { count: available }) : t('productDetail.stockUnavailable'))
      : t('productDetail.addCartLimitExceeded', { count: limit })
    return
  }

  buyNowStore.setItem({
    productId: product.value.id,
    skuId: normalizeSkuId(sku?.id),
    skuCode: String(sku?.sku_code || ''),
    skuSpecValues: (sku?.spec_values && typeof sku.spec_values === 'object') ? sku.spec_values : undefined,
    skuManualStockTotal: normalizeManualStockTotal(sku?.manual_stock_total),
    skuManualStockLocked: normalizeStockNumber(sku?.manual_stock_locked),
    skuManualStockSold: normalizeStockNumber(sku?.manual_stock_sold),
    skuAutoStockAvailable: normalizeStockNumber(sku?.auto_stock_available),
    skuUpstreamStock: normalizeManualStockTotal(sku?.upstream_stock),
    skuStockEnforced: shouldEnforceSkuStock(sku),
    slug: product.value.slug,
    title: product.value.title,
    priceAmount: String(sku?.price_amount || product.value.price_amount || '0.00'),
    image: images.value[0],
    maxPurchaseQuantity: normalizeOptionalLimitNumber(product.value.max_purchase_quantity) ?? undefined,
    purchaseType: product.value.purchase_type,
    fulfillmentType: product.value.fulfillment_type,
    manualFormSchema: product.value.manual_form_schema || {},
    paymentChannelIds: Array.isArray(product.value.payment_channel_ids) && product.value.payment_channel_ids.length > 0 ? product.value.payment_channel_ids : undefined,
    quantity: quantity.value,
  })
  router.push('/checkout?mode=buynow')
}

// Mobile bar price display computed properties
const mobileBarShowMemberPrice = computed(() => {
  if (!selectedSku.value || !hasMemberPrice.value) return false
  const promoPrice = hasSkuPromotionPrice(selectedSku.value) ? getSkuPromotionPriceAmount(selectedSku.value) : selectedSku.value.price_amount
  return selectedSkuMemberPrice.value! < Number(promoPrice)
})
const mobileBarMemberPriceDisplay = computed(() => {
  if (!selectedSkuMemberPrice.value) return ''
  return formatPrice(selectedSkuMemberPrice.value, siteCurrency.value)
})
const mobileBarShowSkuPromotionPrice = computed(() => {
  if (mobileBarShowMemberPrice.value) return false
  return !!selectedSku.value && hasSkuPromotionPrice(selectedSku.value)
})
const mobileBarSkuPromotionPriceDisplay = computed(() => {
  if (!selectedSku.value) return ''
  return formatPrice(getSkuPromotionPriceAmount(selectedSku.value), siteCurrency.value)
})
const mobileBarShowSkuPrice = computed(() => {
  if (mobileBarShowMemberPrice.value || mobileBarShowSkuPromotionPrice.value) return false
  return !!selectedSku.value
})
const mobileBarSkuPriceDisplay = computed(() => {
  if (!selectedSku.value) return ''
  return formatPrice(selectedSku.value.price_amount, siteCurrency.value)
})
const mobileBarShowProductPromotionPrice = computed(() => {
  if (selectedSku.value) return false
  return product.value ? hasPromotionPrice(product.value) : false
})
const mobileBarProductPromotionPriceDisplay = computed(() => {
  if (!product.value) return ''
  return formatPrice(getPromotionPriceAmount(product.value), siteCurrency.value)
})
const mobileBarProductPriceDisplay = computed(() => {
  if (!product.value) return ''
  return formatPrice(product.value.price_amount, siteCurrency.value)
})

const goLogin = () => {
  router.push(`/auth/login?redirect=${encodeURIComponent(route.fullPath)}`)
}

const loadProduct = async () => {
  loading.value = true
  try {
    const slug = route.params.slug as string
    const response = await productAPI.detail(slug)
    product.value = response.data.data || null
    if (images.value.length > 0) {
      currentImage.value = images.value[0] || ''
    }
    syncSelectedSku()
  } catch (error) {
    console.error('Failed to load product:', error)
    product.value = null
    selectedSkuId.value = 0
  } finally {
    loading.value = false
  }
}

const canonicalUrl = computed(() => {
  if (!product.value?.slug) return ''
  return `${window.location.origin}/products/${product.value.slug}`
})

useHead({
  title: () => product.value ? getLocalizedText(product.value.title) : '',
  link: () => {
    if (!canonicalUrl.value) return []
    return [{ rel: 'canonical', href: canonicalUrl.value }]
  },
  meta: () => {
    if (!product.value) return []
    const seoMeta = product.value.seo_meta || {}
    const seoKeywords = getLocalizedText(seoMeta.keywords) || (typeof seoMeta.keywords === 'string' ? seoMeta.keywords : '')
    const seoDescription = getLocalizedText(seoMeta.description) || (typeof seoMeta.description === 'string' ? seoMeta.description : '')
    const tags = []

    if (seoKeywords) tags.push({ name: 'keywords', content: seoKeywords })
    if (seoDescription) tags.push({ name: 'description', content: seoDescription })

    tags.push({ property: 'og:type', content: 'product' })
    if (product.value.title) {
      tags.push({ property: 'og:title', content: getLocalizedText(product.value.title) })
    }
    if (seoDescription) {
      tags.push({ property: 'og:description', content: seoDescription })
    }
    if (images.value && images.value.length > 0) {
      tags.push({ property: 'og:image', content: images.value[0] })
    }
    if (canonicalUrl.value) {
      tags.push({ property: 'og:url', content: canonicalUrl.value })
    }

    tags.push({ name: 'twitter:card', content: 'summary_large_image' })
    if (product.value.title) {
      tags.push({ name: 'twitter:title', content: getLocalizedText(product.value.title) })
    }
    if (seoDescription) {
      tags.push({ name: 'twitter:description', content: seoDescription })
    }
    if (images.value && images.value.length > 0) {
      tags.push({ name: 'twitter:image', content: images.value[0] })
    }

    return tags
  },
  script: () => {
    if (!product.value) return []
    const title = getLocalizedText(product.value.title)
    const seoMeta = product.value.seo_meta || {}
    const description = getLocalizedText(seoMeta.description) || (typeof seoMeta.description === 'string' ? seoMeta.description : '')
    const priceAmount = product.value.price_amount || '0'
    const currency = siteCurrency.value || 'CNY'

    const jsonLd: Record<string, any> = {
      '@context': 'https://schema.org',
      '@type': 'Product',
      name: title,
      url: canonicalUrl.value || window.location.href,
      offers: {
        '@type': 'Offer',
        price: priceAmount,
        priceCurrency: currency,
        availability: product.value.stock_status === 'out_of_stock'
          ? 'https://schema.org/OutOfStock'
          : 'https://schema.org/InStock',
      },
    }
    if (description) jsonLd.description = description
    if (images.value.length > 0) jsonLd.image = images.value
    if (product.value.category?.name) {
      jsonLd.category = getLocalizedText(product.value.category.name)
    }

    return [{
      type: 'application/ld+json',
      innerHTML: JSON.stringify(jsonLd),
    }]
  },
})

onMounted(() => {
  loadProduct()
})

watch(
  () => selectedSkuId.value,
  () => {
    purchaseWarning.value = ''
    quantity.value = 1
  }
)

watch(quantityEffectiveLimit, (limit) => {
  if (limit !== null && quantity.value > limit) {
    quantity.value = Math.max(1, limit)
  }
})
</script>
