<template>
  <div class="home-page min-h-screen theme-page">

    <!-- ==================== LIST MODE ==================== -->
    <template v-if="templateMode === 'list'">
      <!-- Hero Banner (shared with card mode) -->
      <section v-if="showHeroSection" class="relative z-10 border-b theme-border pt-24 pb-10">
        <div class="container max-w-7xl mx-auto px-4">
          <div class="relative overflow-hidden rounded-2xl border theme-panel"
            @touchstart="onBannerTouchStart"
            @touchend="onBannerTouchEnd">
            <Transition name="banner-fade" mode="out-in">
              <img v-if="!bannerLoading && heroImage" :src="heroImage" :key="heroImage" class="absolute inset-0 h-full w-full object-cover" />
            </Transition>
            <div class="absolute inset-0 bg-black/50"></div>
            <div v-if="bannerLoading" class="relative flex min-h-[200px] flex-col justify-between p-5 sm:min-h-[240px] sm:p-6 md:min-h-[320px] md:p-10">
              <div class="space-y-3">
                <div class="h-5 w-24 theme-skeleton rounded-full" style="background: rgba(255,255,255,0.35)"></div>
                <div class="h-8 max-w-3xl theme-skeleton rounded-xl md:h-10" style="background: rgba(255,255,255,0.35)"></div>
                <div class="h-4 max-w-2xl theme-skeleton rounded-lg" style="background: rgba(255,255,255,0.3)"></div>
              </div>
            </div>
            <div v-else class="relative flex min-h-[200px] flex-col justify-between p-5 sm:min-h-[240px] sm:p-6 md:min-h-[320px] md:p-10">
              <div v-if="bannerCount > 1" class="mb-3 flex items-center justify-end gap-2">
                <button type="button"
                  class="inline-flex h-8 w-8 items-center justify-center rounded-full border border-white/30 bg-black/20 text-white transition hover:bg-black/35 md:h-9 md:w-9"
                  @click="handlePrevHeroBanner" :aria-label="t('common.previousBanner')">
                  <svg class="h-3.5 w-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                  </svg>
                </button>
                <button type="button"
                  class="inline-flex h-8 w-8 items-center justify-center rounded-full border border-white/30 bg-black/20 text-white transition hover:bg-black/35 md:h-9 md:w-9"
                  @click="handleNextHeroBanner" :aria-label="t('common.nextBanner')">
                  <svg class="h-3.5 w-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                  </svg>
                </button>
              </div>
              <div class="space-y-2 sm:space-y-3">
                <span class="theme-badge theme-badge-inverse gap-2 text-xs font-semibold uppercase tracking-wider">
                  <span class="h-2 w-2 rounded-full bg-emerald-300"></span>
                  {{ heroBadge }}
                </span>
                <h1 class="max-w-4xl text-xl font-semibold tracking-[-0.02em] text-white sm:text-2xl md:text-3xl">
                  {{ heroTitle }}
                </h1>
                <p class="max-w-3xl text-xs leading-relaxed text-gray-100 sm:text-sm">
                  {{ heroSubtitle }}
                </p>
              </div>
              <div v-if="bannerCount > 1" class="mt-4 flex items-center gap-2">
                <button v-for="(_, bIdx) in banners" :key="`list-dot-${bIdx}`" type="button"
                  class="h-2 rounded-full transition-all"
                  :class="bIdx === currentBannerIndex ? 'w-6 bg-white' : 'w-2 bg-white/45 hover:bg-white/70'"
                  @click="selectHeroBanner(bIdx)"></button>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- Main: Left Categories + Right Product List -->
      <section class="relative z-10 pb-6" :class="showHeroSection ? 'pt-6' : 'pt-24'">
        <div class="container max-w-7xl mx-auto px-4">
          <div class="flex flex-col lg:flex-row gap-6">

            <CategorySidebar
              :categories="listCategoryGroups"
              :selected-category="listSelectedCategory"
              :expanded-parent-ids="listExpandedParentIds"
              :show-drawer="listShowFilterDrawer"
              compact
              @select-category="listSelectCategory"
              @toggle-parent="listToggleParentCategory"
              @update:show-drawer="listShowFilterDrawer = $event"
            />

            <!-- Right: Product List -->
            <main class="flex-1 min-w-0">
              <!-- Search Bar -->
              <div class="relative mb-4">
                <div class="absolute inset-y-0 left-3.5 flex items-center pointer-events-none">
                  <svg class="w-4 h-4 theme-text-muted" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                  </svg>
                </div>
                <input
                  v-model="listSearchQuery"
                  type="text"
                  class="w-full h-10 pl-10 pr-10 rounded-xl border theme-panel text-sm focus:outline-none focus:ring-2 focus:ring-primary/30 theme-text-primary placeholder:theme-text-muted transition-shadow"
                  :placeholder="t('products.searchBoxPlaceholder')"
                  @keydown.enter="listOnSearch"
                />
                <button
                  v-if="listSearchQuery"
                  type="button"
                  class="absolute inset-y-0 right-3 flex items-center theme-text-muted hover:theme-text-primary transition-colors"
                  @click="listClearSearch"
                >
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                  </svg>
                </button>
              </div>

              <!-- Loading Skeleton -->
              <div v-if="listLoading" class="space-y-6">
                <div v-for="i in 3" :key="i">
                  <div class="flex items-center gap-2 mb-3 px-0.5">
                    <div class="h-5 w-5 rounded theme-skeleton"></div>
                    <div class="h-4 w-28 rounded theme-skeleton"></div>
                  </div>
                  <div class="space-y-2">
                    <div v-for="j in 3" :key="j"
                      class="theme-panel rounded-xl border flex items-center h-[72px]">
                      <div class="w-14 h-14 m-2 rounded-lg theme-skeleton flex-shrink-0"></div>
                      <div class="flex-1 px-3 py-2 space-y-2">
                        <div class="h-3.5 w-1/3 rounded theme-skeleton"></div>
                        <div class="h-3 w-1/4 rounded theme-skeleton"></div>
                      </div>
                      <div class="px-4 py-2">
                        <div class="h-4 w-14 rounded theme-skeleton"></div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Grouped Product List -->
              <div v-else-if="listProductGroups.length > 0" class="space-y-6">
                <div v-for="group in listProductGroups" :key="group.categoryId ?? 'uncategorized'">
                  <!-- Category Header -->
                  <div class="flex items-center gap-2 mb-3 px-0.5">
                    <span class="w-1 h-5 rounded-full theme-accent-stick flex-shrink-0"></span>
                    <img v-if="group.categoryIcon" :src="getImageUrl(group.categoryIcon)"
                      :alt="group.categoryName" loading="lazy" class="h-5 w-5 rounded object-cover flex-shrink-0" />
                    <span class="text-sm font-semibold theme-text-primary truncate">{{ group.categoryName }}</span>
                    <span class="text-xs theme-text-muted">({{ group.products.length }})</span>
                  </div>
                  <!-- Products in this category -->
                  <div class="space-y-2">
                    <ProductListItem
                      v-for="(product, idx) in group.products"
                      :key="product.id"
                      :product="product"
                      :index="idx"
                      :animation-step="20"
                      @click="goToProduct"
                      @quick-buy="openQuickBuy"
                    />
                  </div>
                </div>

                <PaginationNav
                  :current-page="listCurrentPage"
                  :total-pages="listTotalPages"
                  compact
                  @change-page="listChangePage"
                />
              </div>

              <!-- Empty State -->
              <div v-else class="text-center py-16 border theme-panel-soft rounded-2xl backdrop-blur-sm">
                <svg class="w-16 h-16 mx-auto theme-text-muted mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
                    d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2.586a1 1 0 00-.707.293l-2.414 2.414a1 1 0 01-.707.293h-3.172a1 1 0 01-.707-.293l-2.414-2.414A1 1 0 006.586 13H4" />
                </svg>
                <p class="theme-text-muted text-lg">{{ t('products.empty') }}</p>
              </div>
            </main>
          </div>
        </div>
      </section>
    </template>

    <!-- ==================== CARD MODE (default) ==================== -->
    <template v-else>
    <section v-if="showHeroSection" class="relative z-10 border-b theme-border pt-24 pb-10">
      <div class="container max-w-7xl mx-auto px-4 sm:px-6">
        <div class="relative overflow-hidden rounded-2xl border theme-panel"
          @touchstart="onBannerTouchStart"
          @touchend="onBannerTouchEnd">
          <!-- Banner image with fade transition -->
          <Transition name="banner-fade" mode="out-in">
            <img v-if="!bannerLoading && heroImage" :src="heroImage" :key="heroImage" class="absolute inset-0 h-full w-full object-cover" />
          </Transition>
          <div class="absolute inset-0 bg-black/50"></div>

            <div v-if="bannerLoading" class="relative flex min-h-[260px] flex-col justify-between p-5 sm:min-h-[320px] sm:p-6 md:min-h-[420px] md:p-12">
            <div class="mb-4 flex items-center justify-end">
              <span class="theme-badge theme-badge-inverse text-xs font-medium">
                {{ t('common.loading') }}
              </span>
            </div>

            <div class="space-y-4">
              <div class="h-6 w-28 theme-skeleton rounded-full" style="background: rgba(255,255,255,0.35)"></div>
              <div class="h-10 max-w-4xl theme-skeleton rounded-xl md:h-14" style="background: rgba(255,255,255,0.35)"></div>
              <div class="h-5 max-w-3xl theme-skeleton rounded-lg" style="background: rgba(255,255,255,0.3)"></div>
            </div>

            <div class="flex flex-wrap items-center gap-3 pt-6">
              <div class="h-11 w-36 theme-skeleton rounded-lg" style="background: rgba(255,255,255,0.35)"></div>
              <div class="h-11 w-28 theme-skeleton rounded-lg" style="background: rgba(255,255,255,0.25)"></div>
            </div>
          </div>

          <div v-else class="relative flex min-h-[260px] flex-col justify-between p-5 sm:min-h-[320px] sm:p-6 md:min-h-[420px] md:p-12">
            <div v-if="bannerCount > 1" class="mb-4 flex items-center justify-end gap-2">
              <button
                type="button"
                class="inline-flex h-9 w-9 items-center justify-center rounded-full border border-white/30 bg-black/20 text-white transition hover:bg-black/35 md:h-10 md:w-10"
                @click="handlePrevHeroBanner"
                :aria-label="t('common.previousBanner')"
              >
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                </svg>
              </button>
              <button
                type="button"
                class="inline-flex h-9 w-9 items-center justify-center rounded-full border border-white/30 bg-black/20 text-white transition hover:bg-black/35 md:h-10 md:w-10"
                @click="handleNextHeroBanner"
                :aria-label="t('common.nextBanner')"
              >
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                </svg>
              </button>
            </div>

            <div class="space-y-3 sm:space-y-4">
              <span class="theme-badge theme-badge-inverse gap-2 text-xs font-semibold uppercase tracking-wider">
                <span class="h-2 w-2 rounded-full bg-emerald-300"></span>
                {{ heroBadge }}
              </span>
              <h1 class="max-w-4xl text-2xl font-semibold tracking-[-0.02em] text-white sm:text-3xl md:text-[2.85rem]">
                {{ heroTitle }}
              </h1>
              <p class="max-w-3xl text-xs leading-relaxed text-gray-100 sm:text-sm md:text-base">
                {{ heroSubtitle }}
              </p>
            </div>

            <div class="flex flex-wrap items-center gap-3 pt-5 sm:pt-6">
              <button
                type="button"
                @click="goToHeroLink"
                class="inline-flex min-h-[40px] items-center gap-2 rounded-lg bg-white px-4 py-2.5 text-sm font-semibold text-gray-900 transition hover:scale-105 sm:min-h-[44px] sm:px-5 sm:py-3"
              >
                {{ heroPrimaryButtonText }}
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 8l4 4m0 0l-4 4m4-4H3" />
                </svg>
              </button>
              <router-link
                v-if="!hasHeroLink"
                to="/products"
                class="inline-flex min-h-[40px] items-center rounded-lg border border-white/30 px-4 py-2.5 text-sm font-medium text-white transition hover:border-white hover:bg-white/10 sm:min-h-[44px] sm:px-5 sm:py-3"
              >
                {{ t('home.featured.viewAll') }}
              </router-link>
            </div>

            <div v-if="bannerCount > 1" class="mt-5 flex items-center gap-2">
              <button
                v-for="(_, index) in banners"
                :key="`hero-dot-${index}`"
                type="button"
                class="h-2.5 rounded-full transition-all"
                :class="index === currentBannerIndex ? 'w-7 bg-white' : 'w-2.5 bg-white/45 hover:bg-white/70'"
                @click="selectHeroBanner(index)"
                :aria-label="t('common.switchBanner', { n: index + 1 })"
              ></button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section
      v-if="promoProducts.length > 0"
      id="promotion"
      class="relative z-10 pb-14"
      :class="showHeroSection ? 'pt-14' : 'pt-32 md:pt-36'"
    >
      <div class="container max-w-7xl mx-auto px-4">
        <div class="home-section-header home-section-header-promo mb-6">
          <div class="home-section-header-row">
            <div class="home-section-copy">
              <h2 class="theme-section-heading home-section-heading-accent text-[1.9rem] md:text-[2.3rem]">{{ t('home.promo.title') }}</h2>
              <p class="home-section-description">{{ t('home.promo.description') }}</p>
            </div>
          <router-link
            v-if="!hasHeroLink"
            to="/products"
            class="home-section-action"
          >
            {{ t('home.promo.viewMore') }}
          </router-link>
          </div>
          <div class="home-section-rule"></div>
        </div>

        <PromotionSection
          :products="promoProducts"
          @click="goToProduct"
        />
      </div>
    </section>

    <section
      id="featured"
      class="relative z-10 pb-14"
      :class="promoProducts.length > 0 ? 'pt-0' : (showHeroSection ? 'pt-14' : 'pt-32 md:pt-36')"
    >
      <div class="container max-w-7xl mx-auto px-4">
        <div class="home-section-header home-section-header-featured mb-6">
          <div class="home-section-header-row">
            <div class="home-section-copy">
              <h2 class="theme-section-heading home-section-heading-accent text-[1.9rem] md:text-[2.3rem]">{{ t('home.featured.title') }}</h2>
              <p class="home-section-description">{{ t('home.featured.description') }}</p>
            </div>
            <router-link
              v-if="!hasHeroLink"
              to="/products"
              class="home-section-action"
            >
              <span>{{ t('home.featured.viewAll') }}</span>
            </router-link>
          </div>
          <div class="home-section-rule"></div>
        </div>

        <div v-if="products.length > 0" class="grid grid-cols-2 gap-3 md:gap-4 md:grid-cols-3 lg:grid-cols-4">
          <ProductCard
            v-for="(product, idx) in products"
            :key="product.id"
            :product="product"
            :index="idx"
            :animation-step="60"
            @click="goToProduct"
            @quick-buy="openQuickBuy"
          />
        </div>
        <div v-else class="rounded-2xl border border-dashed theme-border py-16 text-center theme-text-muted theme-slide-up">
          <svg class="mx-auto h-16 w-16 mb-4 theme-text-muted opacity-50" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2.586a1 1 0 00-.707.293l-2.414 2.414a1 1 0 01-.707.293h-3.172a1 1 0 01-.707-.293l-2.414-2.414A1 1 0 006.586 13H4" />
          </svg>
          {{ t('home.featured.empty') }}
        </div>
      </div>
    </section>

    <template v-if="latestSectionVisible">
    <section class="relative z-10 py-12">
      <div class="container max-w-7xl mx-auto px-4">
        <div class="home-section-header home-section-header-latest mb-4">
          <div class="home-section-header-row">
            <div class="home-section-copy">
              <h2 class="theme-section-heading home-section-heading-accent text-[1.9rem] md:text-[2.3rem]">{{ t('home.latest.title') }}</h2>
              <p class="home-section-description">{{ t('home.latest.description') }}</p>
            </div>
          </div>
          <div class="home-section-rule"></div>
        </div>

        <div
          v-if="blogEnabled || noticeEnabled"
          class="home-latest-grid"
          :class="{ 'home-latest-grid-single': blogEnabled !== noticeEnabled }"
        >
          <section v-if="blogEnabled" class="home-latest-stream home-latest-stream-blog">
            <div class="home-latest-stream-head">
              <div class="home-latest-stream-cover" :class="{ 'home-latest-stream-cover-placeholder': !getLatestLeadImage(blogPosts) }">
                <img
                  v-if="getLatestLeadImage(blogPosts)"
                  :src="getLatestLeadImage(blogPosts)"
                  :alt="t('nav.blog')"
                  class="home-latest-stream-cover-image"
                />
                <div class="home-latest-stream-cover-overlay"></div>
                <div class="home-latest-stream-cover-badge">
                  {{ t('nav.blog') }}
                </div>
              </div>

              <div class="home-latest-stream-summary">
                <div class="home-latest-stream-summary-top">
                  <h3 class="home-latest-stream-title">{{ t('nav.blog') }}</h3>
                  <p class="home-latest-stream-description">{{ t('blog.subtitle') }}</p>
                </div>

                <div class="home-latest-stream-meta">
                  <span class="home-latest-stream-count">
                    <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9a2 2 0 00-2-2h-2" />
                    </svg>
                    <span>{{ t('home.latest.count', { count: blogPosts.length }) }}</span>
                  </span>
                  <router-link to="/blog" class="home-latest-stream-more">
                    {{ t('home.latest.moreEntries') }}
                  </router-link>
                </div>
              </div>
            </div>

            <div v-if="blogPosts.length > 0" class="home-latest-stream-list">
              <article
                v-for="post in blogPosts"
                :key="`blog-${post.id}`"
                class="home-latest-stream-item"
                @click="goToPost(post.slug)"
              >
                <div class="home-latest-stream-item-main">
                  <span class="home-latest-stream-item-dot"></span>
                  <h4 class="home-latest-stream-item-title line-clamp-1">{{ getLocalizedText(post.title) }}</h4>
                </div>
                <div v-if="formatDate(post.created_at)" class="home-latest-stream-item-time">
                  <svg class="h-3 w-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                  </svg>
                  <span>{{ formatDate(post.created_at) }}</span>
                </div>
              </article>
            </div>
            <div v-else class="home-latest-empty">
              {{ t('blog.empty') }}
            </div>
          </section>

          <section v-if="noticeEnabled" class="home-latest-stream home-latest-stream-notice">
            <div class="home-latest-stream-head">
              <div class="home-latest-stream-cover home-latest-stream-cover-notice" :class="{ 'home-latest-stream-cover-placeholder': !getLatestLeadImage(noticePosts) }">
                <img
                  v-if="getLatestLeadImage(noticePosts)"
                  :src="getLatestLeadImage(noticePosts)"
                  :alt="t('nav.notice')"
                  class="home-latest-stream-cover-image"
                />
                <div class="home-latest-stream-cover-overlay"></div>
                <div class="home-latest-stream-cover-badge home-latest-stream-cover-badge-notice">
                  {{ t('nav.notice') }}
                </div>
              </div>

              <div class="home-latest-stream-summary">
                <div class="home-latest-stream-summary-top">
                  <h3 class="home-latest-stream-title">{{ t('nav.notice') }}</h3>
                  <p class="home-latest-stream-description">{{ t('notice.subtitle') }}</p>
                </div>

                <div class="home-latest-stream-meta">
                  <span class="home-latest-stream-count">
                    <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
                    </svg>
                    <span>{{ t('home.latest.count', { count: noticePosts.length }) }}</span>
                  </span>
                  <router-link to="/notice" class="home-latest-stream-more">
                    {{ t('home.latest.moreEntries') }}
                  </router-link>
                </div>
              </div>
            </div>

            <div v-if="noticePosts.length > 0" class="home-latest-stream-list">
              <article
                v-for="post in noticePosts"
                :key="`notice-${post.id}`"
                class="home-latest-stream-item"
                @click="goToPost(post.slug)"
              >
                <div class="home-latest-stream-item-main">
                  <span class="home-latest-stream-item-dot home-latest-stream-item-dot-notice"></span>
                  <h4 class="home-latest-stream-item-title line-clamp-1">{{ getLocalizedText(post.title) }}</h4>
                </div>
                <div v-if="formatDate(post.created_at)" class="home-latest-stream-item-time">
                  <svg class="h-3 w-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                  </svg>
                  <span>{{ formatDate(post.created_at) }}</span>
                </div>
              </article>
            </div>
            <div v-else class="home-latest-empty">
              {{ t('notice.empty') }}
            </div>
          </section>
        </div>
        <div v-else class="rounded-2xl border border-dashed theme-border py-12 text-center theme-text-muted">
          {{ t('blog.empty') }}
        </div>
      </div>
    </section>
    </template>
    </template>

    <ProductQuickBuy
      v-if="quickBuyProduct"
      :product="quickBuyProduct"
      :visible="quickBuyVisible"
      @update:visible="quickBuyVisible = $event"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { postAPI, productAPI } from '../api'
import { getImageUrl } from '../utils/image'
import { useLocalized, useProductLabels } from '../composables/useProduct'
import { useBannerCarousel } from '../composables/useBannerCarousel'
import { useProductList } from '../composables/useProductList'
import { useProductListGroups } from '../composables/useProductListGroups'
import { useAppStore } from '../stores/app'
import ProductCard from '../components/ProductCard.vue'
import PromotionSection from '../components/PromotionSection.vue'
import ProductListItem from '../components/ProductListItem.vue'
import ProductQuickBuy from '../components/ProductQuickBuy.vue'
import CategorySidebar from '../components/CategorySidebar.vue'
import PaginationNav from '../components/PaginationNav.vue'

const router = useRouter()
const { t } = useI18n()
const { getLocalizedText } = useLocalized()
const { hasPromotionPrice, hasPromotionRules } = useProductLabels()
const appStore = useAppStore()

const templateMode = computed(() => appStore.config?.template_mode || 'card')
const navBuiltin = computed(() => (appStore.config?.nav_config as { builtin?: Record<string, boolean> } | undefined)?.builtin)
const blogEnabled = computed(() => navBuiltin.value?.blog !== false)
const noticeEnabled = computed(() => navBuiltin.value?.notice !== false)
const latestSectionVisible = computed(() => blogEnabled.value || noticeEnabled.value)

// ==================== Shared State ====================
const products = ref<any[]>([])
const promoProducts = ref<any[]>([])
const blogPosts = ref<any[]>([])
const noticePosts = ref<any[]>([])
const quickBuyProduct = ref<any>(null)
const quickBuyVisible = ref(false)

const openQuickBuy = (product: any) => {
  quickBuyProduct.value = product
  quickBuyVisible.value = true
}

// ==================== Banner Carousel ====================
const {
  banners,
  bannerLoading,
  currentBannerIndex,
  bannerCount,
  showHeroSection,
  heroImage,
  heroBadge,
  heroTitle,
  heroSubtitle,
  hasHeroLink,
  heroPrimaryButtonText,
  loadBanners,
  handleNextHeroBanner,
  handlePrevHeroBanner,
  selectHeroBanner,
  goToHeroLink,
  onBannerTouchStart,
  onBannerTouchEnd,
  stopHeroAutoPlay,
} = useBannerCarousel()

// ==================== List Mode ====================
const {
  loading: listLoading,
  products: listProducts,
  selectedCategory: listSelectedCategory,
  searchQuery: listSearchQuery,
  currentPage: listCurrentPage,
  totalPages: listTotalPages,
  showFilterDrawer: listShowFilterDrawer,
  expandedParentIds: listExpandedParentIds,
  categoryGroups: listCategoryGroups,
  categoryMap: listCategoryMap,
  selectCategory: listSelectCategory,
  toggleParentCategory: listToggleParentCategory,
  changePage: listChangePage,
  clearSearch: listClearSearch,
  onSearch: listOnSearch,
  initialize: listInitialize,
  cleanup: listCleanup,
} = useProductList({ pageSize: 20, homeRouteName: 'home' })

const listProductGroups = useProductListGroups(listProducts, listCategoryMap)

// ==================== Card Mode ====================
const formatDate = (dateString: string) => {
  if (!dateString) return ''

  const date = new Date(dateString)
  if (Number.isNaN(date.getTime())) return ''

  return `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`
}

const goToProduct = (slug: string) => {
  router.push(`/products/${slug}`)
}

const goToPost = (slug: string) => {
  router.push(`/blog/${slug}`)
}

const getLatestLeadImage = (items: any[]) => {
  const lead = items.find((item) => String(item?.thumbnail || '').trim() !== '')
  return lead ? getImageUrl(lead.thumbnail) : ''
}

const isPromotionProduct = (product: any) => {
  return hasPromotionPrice(product) || hasPromotionRules(product)
}

const loadPromotionProducts = async () => {
  try {
    const response = await productAPI.list({ page: 1, page_size: 48 })
    const items = response.data.data || []
    promoProducts.value = items.filter(isPromotionProduct).slice(0, 8)
  } catch (error) {
    promoProducts.value = []
    console.error('Failed to load promotion products:', error)
  }
}

const loadFeaturedProducts = async () => {
  try {
    const response = await productAPI.list({ page: 1, page_size: 48 })
    const items = response.data.data || []
    products.value = items.filter((product: any) => !isPromotionProduct(product)).slice(0, 12)
  } catch (error) {
    console.error('Failed to load products:', error)
  }
}

const loadPostColumn = async (type: 'blog' | 'notice') => {
  const response = await postAPI.list({
    type,
    page: 1,
    page_size: 5,
  })
  return response.data.data || []
}

const loadLatestPosts = async () => {
  if (!latestSectionVisible.value) return

  const tasks: Promise<any[]>[] = []
  const taskKeys: Array<'blog' | 'notice'> = []

  if (blogEnabled.value) {
    taskKeys.push('blog')
    tasks.push(loadPostColumn('blog'))
  } else {
    blogPosts.value = []
  }

  if (noticeEnabled.value) {
    taskKeys.push('notice')
    tasks.push(loadPostColumn('notice'))
  } else {
    noticePosts.value = []
  }

  try {
    const results = await Promise.allSettled(tasks)
    results.forEach((result, index) => {
      const key = taskKeys[index]
      if (result.status === 'fulfilled') {
        if (key === 'blog') blogPosts.value = result.value
        if (key === 'notice') noticePosts.value = result.value
        return
      }

      if (key === 'blog') blogPosts.value = []
      if (key === 'notice') noticePosts.value = []
      console.error(`Failed to load ${key} posts:`, result.reason)
    })
  } catch (error) {
    console.error('Failed to load latest posts:', error)
  }
}

// ==================== Lifecycle ====================
onMounted(async () => {
  if (templateMode.value === 'list') {
    await Promise.all([loadBanners(), listInitialize()])
  } else {
    await Promise.all([loadBanners(), loadPromotionProducts(), loadFeaturedProducts(), loadLatestPosts()])
  }
})

onUnmounted(() => {
  stopHeroAutoPlay()
  listCleanup()
})
</script>

<style scoped>
.banner-fade-enter-active,
.banner-fade-leave-active {
  transition: opacity 300ms ease;
}
.banner-fade-enter-from,
.banner-fade-leave-to {
  opacity: 0;
}

.promo-more-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  min-height: 2.6rem;
  padding: 0.5rem 1.15rem;
  border-radius: 0.5rem;
  color: #fff;
  background-color: #000;
  box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.14);
  font-size: 0.95rem;
  font-weight: 600;
  line-height: 1.15;
  text-decoration: none;
  white-space: nowrap;
  word-break: keep-all;
  transition: transform 220ms ease, background-color 220ms ease, box-shadow 220ms ease, opacity 220ms ease;
}

.promo-more-btn:hover,
.promo-more-btn:focus-visible {
  background-color: #161616;
  color: #fff;
  box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.36), 0 0.45rem 1rem -0.45rem rgba(0, 0, 0, 0.32);
}

.promo-more-btn:active {
  transform: scale(0.97);
  opacity: 0.92;
}

.home-section-header {
  position: relative;
}

.home-section-header-row {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 1rem;
}

.home-section-rule {
  margin-top: 0.9rem;
  height: 1px;
  width: 100%;
  background:
    linear-gradient(
      90deg,
      color-mix(in oklab, var(--ui-border) 88%, transparent) 0%,
      color-mix(in oklab, var(--ui-border) 54%, transparent) 32%,
      transparent 100%
    );
}

:global(.dark) .home-section-rule {
  background:
    linear-gradient(
      90deg,
      color-mix(in oklab, var(--ui-border) 92%, transparent) 0%,
      color-mix(in oklab, var(--ui-border) 62%, transparent) 34%,
      transparent 100%
    );
}

.home-section-copy {
  position: relative;
  max-width: 36rem;
}

.home-section-heading-accent {
  position: relative;
  display: inline-block;
  isolation: isolate;
}

.home-section-heading-accent::after {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0.14rem;
  z-index: -1;
  height: 0.64rem;
  border-radius: 0.18rem;
  transform: rotate(-1.3deg);
  transform-origin: left center;
  background: linear-gradient(90deg, #22d3ee 0%, #e879f9 52%, #facc15 100%);
  opacity: 0.92;
}

.home-section-description {
  margin-top: 0.24rem;
  max-width: 32rem;
  font-size: 0.81rem;
  line-height: 1.45;
  color: var(--ui-text-secondary);
}

.home-section-description-compact {
  max-width: 28rem;
}

.home-section-action {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.42rem;
  flex-shrink: 0;
  min-height: 2.3rem;
  padding: 0 0.88rem;
  border-radius: 999px;
  border: 1px solid color-mix(in oklab, var(--ui-border) 80%, white 20%);
  background: transparent;
  color: var(--ui-text-primary);
  font-size: 0.76rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  text-decoration: none;
  white-space: nowrap;
  transition:
    transform 0.2s ease,
    border-color 0.2s ease,
    background-color 0.2s ease,
    color 0.2s ease;
}

.home-section-action:hover {
  transform: translateY(-1px);
  border-color: color-mix(in oklab, var(--ui-accent) 26%, var(--ui-border));
  background: color-mix(in oklab, var(--ui-bg-soft) 90%, transparent);
  color: color-mix(in oklab, var(--ui-accent) 52%, var(--ui-text-primary));
}

.home-channel-links {
  position: relative;
  z-index: 1;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: flex-end;
  gap: 0.7rem;
}

.home-channel-link {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  min-height: 2.5rem;
  padding: 0 0.92rem;
  border-radius: 999px;
  border: 1px solid color-mix(in oklab, var(--ui-border) 74%, white 26%);
  background: color-mix(in oklab, var(--ui-bg-soft) 86%, transparent);
  color: var(--ui-text-secondary);
  font-size: 0.82rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  text-decoration: none;
  transition:
    transform 0.2s ease,
    color 0.2s ease,
    border-color 0.2s ease,
    background-color 0.2s ease,
    box-shadow 0.2s ease;
}

.home-channel-link:hover {
  transform: translateY(-1px);
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-accent) 20%, var(--ui-border));
  background: color-mix(in oklab, var(--ui-bg-overlay-strong) 88%, transparent);
  box-shadow: 0 16px 24px -24px rgba(15, 23, 42, 0.22);
}

.home-channel-link-icon {
  width: 0.92rem;
  height: 0.92rem;
  flex-shrink: 0;
  opacity: 0.8;
}

.home-latest-grid {
  display: grid;
  grid-template-columns: repeat(1, minmax(0, 1fr));
  gap: 1.25rem;
}

.home-latest-grid-single {
  max-width: 42rem;
}

.home-latest-stream {
  position: relative;
  overflow: hidden;
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  border-radius: 0.92rem;
  padding: 0.9rem 0.9rem 0.4rem;
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 94%, white 6%),
      color-mix(in oklab, var(--ui-bg-overlay) 96%, transparent)
    );
  box-shadow: var(--ui-shadow-1);
}

:global(.dark) .home-latest-stream {
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 97%, black 3%),
      color-mix(in oklab, var(--ui-bg-muted) 94%, black 6%)
    );
  border-color: color-mix(in oklab, var(--ui-border) 86%, white 14%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 28px 40px -36px rgba(0, 0, 0, 0.4);
}

.home-latest-stream-head {
  display: grid;
  grid-template-columns: minmax(0, 1.15fr) minmax(0, 1fr);
  gap: 0.8rem;
  align-items: stretch;
  margin-bottom: 0.8rem;
}

.home-latest-stream-cover {
  position: relative;
  min-height: 8.9rem;
  overflow: hidden;
  border-radius: 1rem;
  background:
    linear-gradient(135deg, rgba(37, 99, 235, 0.92), rgba(56, 189, 248, 0.86)),
    linear-gradient(180deg, rgba(255, 255, 255, 0.18), transparent);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.18);
}

.home-latest-stream-cover-notice {
  background:
    linear-gradient(135deg, rgba(59, 130, 246, 0.88), rgba(14, 165, 233, 0.74)),
    linear-gradient(180deg, rgba(255, 255, 255, 0.18), transparent);
}

.home-latest-stream-cover-placeholder::before,
.home-latest-stream-cover-placeholder::after {
  content: '';
  position: absolute;
  border-radius: 999px;
  filter: blur(4px);
  opacity: 0.8;
}

.home-latest-stream-cover-placeholder::before {
  width: 7rem;
  height: 7rem;
  right: -1rem;
  top: -1rem;
  background: rgba(255, 255, 255, 0.24);
}

.home-latest-stream-cover-placeholder::after {
  width: 10rem;
  height: 10rem;
  left: -2.5rem;
  bottom: -3rem;
  background: rgba(255, 255, 255, 0.14);
}

.home-latest-stream-cover-image {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.home-latest-stream-cover-overlay {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(180deg, rgba(15, 23, 42, 0.1), rgba(15, 23, 42, 0.46)),
    linear-gradient(90deg, rgba(255, 255, 255, 0.08), transparent 55%);
}

.home-latest-stream-cover-badge {
  position: absolute;
  top: 0.68rem;
  left: 0.68rem;
  z-index: 1;
  display: inline-flex;
  align-items: center;
  min-height: 1.58rem;
  padding: 0 0.58rem;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-size: 0.66rem;
  font-weight: 700;
  letter-spacing: 0.02em;
  backdrop-filter: blur(10px);
}

.home-latest-stream-cover-badge-notice {
  background: rgba(15, 23, 42, 0.22);
}

.home-latest-stream-summary {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-width: 0;
  align-self: stretch;
}

.home-latest-stream-summary-top {
  display: block;
  padding-top: 0.35rem;
}

.home-latest-stream-title {
  color: var(--ui-text-primary);
  font-size: clamp(1.28rem, 1.08rem + 0.42vw, 1.6rem);
  font-weight: 800;
  line-height: 1.1;
  letter-spacing: -0.03em;
}

.home-latest-stream-description {
  margin-top: 0.48rem;
  color: var(--ui-text-secondary);
  font-size: 0.84rem;
  line-height: 1.5;
}

.home-latest-stream-meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.7rem;
  margin-top: 0.82rem;
}

.home-latest-stream-count {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  color: var(--ui-text-muted);
  font-size: 0.76rem;
  font-weight: 600;
  letter-spacing: -0.01em;
}

.home-latest-stream-more {
  display: inline-flex;
  align-items: center;
  border-radius: 999px;
  min-height: 2.05rem;
  padding: 0 0.78rem;
  border: 1px solid color-mix(in oklab, var(--ui-border) 76%, white 24%);
  background: color-mix(in oklab, var(--ui-bg-soft) 88%, transparent);
  color: color-mix(in oklab, var(--ui-accent) 68%, var(--ui-text-primary));
  font-size: 0.73rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  text-decoration: none;
  transition:
    transform 0.2s ease,
    border-color 0.2s ease,
    background-color 0.2s ease,
    color 0.2s ease;
}

.home-latest-stream-more:hover {
  transform: translateY(-1px);
  border-color: color-mix(in oklab, var(--ui-accent) 28%, var(--ui-border));
  color: var(--ui-text-primary);
}

.home-latest-stream-list {
  display: grid;
}

.home-latest-stream-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.56rem;
  padding: 0.32rem 0.04rem;
  cursor: pointer;
  transition: transform 0.2s ease, color 0.2s ease;
}

.home-latest-stream-item:hover,
.home-latest-stream-item:active {
  transform: translateX(2px);
}

.home-latest-stream-item:hover .home-latest-stream-item-title,
.home-latest-stream-item:active .home-latest-stream-item-title {
  color: color-mix(in oklab, rgb(239 68 68) 78%, var(--ui-text-primary));
}

.home-latest-stream-item-main {
  display: flex;
  align-items: center;
  min-width: 0;
  gap: 0.38rem;
  flex: 1;
}

.home-latest-stream-item-dot {
  width: 0.46rem;
  height: 0.46rem;
  flex-shrink: 0;
  border: 1.55px solid rgba(244, 63, 94, 0.78);
  border-radius: 999px;
  background: transparent;
}

.home-latest-stream-item-dot-notice {
  border-color: rgba(59, 130, 246, 0.8);
}

.home-latest-stream-item-title {
  color: var(--ui-text-primary);
  font-size: 0.84rem;
  font-weight: 600;
  line-height: 1.5;
  letter-spacing: -0.015em;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.home-latest-stream-item-time {
  display: inline-flex;
  align-items: center;
  gap: 0.28rem;
  flex-shrink: 0;
  color: var(--ui-text-muted);
  font-size: 0.72rem;
  font-weight: 500;
  white-space: nowrap;
}

.home-latest-empty {
  border-top: 1px dashed color-mix(in oklab, var(--ui-border) 72%, transparent);
  padding: 1.1rem 0.25rem 0.7rem;
  color: var(--ui-text-muted);
  font-size: 0.9rem;
  text-align: center;
}

@media (max-width: 767px) {
  .home-section-header-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.48rem;
  }

  .home-section-header-featured .home-section-header-row {
    flex-direction: row;
    align-items: flex-end;
    justify-content: space-between;
    gap: 0.75rem;
  }

  .home-section-header-promo .home-section-header-row {
    flex-direction: row;
    align-items: flex-end;
    justify-content: space-between;
    gap: 0.75rem;
  }

  .home-section-header-featured .home-section-copy {
    flex: 1;
    min-width: 0;
  }

  .home-section-header-promo .home-section-copy {
    flex: 1;
    min-width: 0;
  }

  .home-section-action {
    width: auto;
    align-self: flex-start;
  }

  .home-section-header-featured .home-section-action {
    align-self: flex-end;
    margin-bottom: 0.08rem;
  }

  .home-section-header-promo .home-section-action {
    align-self: flex-end;
    margin-bottom: 0.08rem;
  }

  .home-section-rule {
    margin-top: 0.72rem;
  }

  .home-section-heading-accent::after {
    bottom: 0.08rem;
    height: 0.52rem;
  }

  .home-channel-links {
    justify-content: flex-start;
  }

  .home-latest-stream {
    border-radius: 0.82rem;
    padding: 0.82rem 0.82rem 0.28rem;
  }

  .home-latest-stream-head {
    grid-template-columns: 1fr;
  }

  .home-latest-stream-cover {
    min-height: 7.6rem;
  }

  .home-latest-stream-summary-top {
    display: none;
  }

  .home-latest-stream-meta {
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    row-gap: 0.5rem;
  }

  .home-latest-stream-more {
    width: auto;
    justify-content: center;
    margin-left: auto;
  }

  .home-latest-stream-item {
    flex-direction: row;
    align-items: center;
    gap: 0.75rem;
  }

  .home-latest-stream-item-time {
    padding-left: 0;
    margin-left: auto;
  }
}

@media (min-width: 768px) {
  .home-latest-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
</style>
