<template>
  <nav
    class="fixed top-0 left-0 right-0 z-50 theme-panel-soft border-b theme-border backdrop-blur-md transition-all"
    :class="scrolled ? 'py-2 shadow-lg' : 'py-4'"
    :style="{ transitionDuration: 'var(--ui-duration-normal)' }">
    <div class="hidden lg:flex container mx-auto px-4 items-center gap-6">
      <!-- Brand -->
      <router-link to="/" class="apple-brand group relative shrink-0" :title="brandSiteName">
        <img src="/logo.svg" alt="" class="apple-brand-logo" aria-hidden="true" />
        <span class="apple-brand-copy">
          <span class="apple-brand-name">{{ brandSiteName }}</span>
        </span>
      </router-link>

      <!-- Desktop Menu -->
      <div class="hidden lg:flex min-w-0 flex-1 justify-center">
        <div
          ref="desktopNavShell"
          class="apple-nav-shell scrollbar-hide"
          @mouseleave="clearNavHover"
          @focusout="handleDesktopNavFocusOut"
        >
          <span
            class="apple-nav-indicator"
            :class="{ 'apple-nav-indicator-visible': navIndicatorVisible }"
            :style="navIndicatorStyle"
            aria-hidden="true"
          />
          <template v-for="item in menuItems" :key="item.key">
            <router-link
              v-if="item.type === 'route'"
              :to="item.path"
              class="apple-nav-link"
              :class="{ 'apple-nav-link-active': isRouteItemActive(item) }"
              :data-nav-key="item.key"
              @mouseenter="setNavHover(item.key)"
              @focus="setNavHover(item.key)"
            >
              <svg
                v-if="item.icon"
                class="apple-nav-link-icon"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" :d="item.icon" />
              </svg>
              <span>{{ item.label.startsWith('nav.') ? t(item.label) : item.label }}</span>
            </router-link>
            <a
              v-else
              :href="item.path"
              :target="item.target"
              rel="noopener noreferrer"
              class="apple-nav-link"
              :class="{ 'apple-nav-link-active': isRouteItemActive(item) }"
              :data-nav-key="item.key"
              @mouseenter="setNavHover(item.key)"
              @focus="setNavHover(item.key)"
            >
              <svg
                v-if="item.icon"
                class="apple-nav-link-icon"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" :d="item.icon" />
              </svg>
              <span>{{ item.label }}</span>
            </a>
          </template>
        </div>
      </div>

      <!-- Right Side Actions -->
      <div class="ml-auto flex items-center shrink-0 gap-2 lg:gap-3">
        <div class="hidden lg:flex items-center gap-2">
          <router-link
            v-if="!userAuthStore.isAuthenticated"
            to="/guest/orders"
            class="apple-action-pill apple-action-pill-secondary apple-action-icon"
            :aria-label="t('navbar.guestOrders')"
            :title="t('navbar.guestOrders')"
          >
            <svg class="w-[1.32rem] h-[1.32rem] shrink-0 opacity-95" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
            </svg>
          </router-link>
          <router-link
            v-if="!userAuthStore.isAuthenticated"
            to="/auth/login"
            class="apple-action-pill apple-action-pill-primary"
          >
            <svg class="w-4 h-4 shrink-0 opacity-90" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" d="M11 16l-4-4m0 0l4-4m-4 4h14m-5 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h7a3 3 0 013 3v1" />
            </svg>
            <span>{{ t('navbar.login') }}</span>
          </router-link>
          <div
            v-if="userAuthStore.isAuthenticated"
            class="apple-user-menu user-menu"
            @mouseenter="openUserMenu"
            @mouseleave="scheduleCloseUserMenu"
            @focusin="openUserMenu"
            @focusout="handleUserMenuFocusOut"
          >
            <router-link
              to="/me"
              class="apple-action-pill apple-action-pill-primary"
            >
              <svg class="w-4 h-4 shrink-0 opacity-90" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
              </svg>
              <span>{{ t('navbar.personalCenter') }}</span>
            </router-link>

            <div
              v-if="showUserMenu"
              class="apple-user-menu-panel absolute right-0 top-full z-50 theme-panel-strong border"
              @mouseenter="openUserMenu"
              @mouseleave="scheduleCloseUserMenu"
            >
              <div class="apple-user-card">
                <div class="apple-user-card-head">
                  <div class="apple-user-avatar" aria-hidden="true">{{ userInitial }}</div>
                  <div class="min-w-0 flex-1">
                    <div class="apple-user-name-row">
                      <span class="apple-user-name">{{ userDisplayName }}</span>
                      <span class="apple-user-level">
                        <img
                          v-if="isImagePath(currentLevelIcon)"
                          :src="getImageUrl(currentLevelIcon)"
                          class="apple-user-level-icon-image"
                          alt=""
                        />
                        <span v-else-if="currentLevelIcon" class="apple-user-level-icon-text">{{ currentLevelIcon }}</span>
                        <span>{{ currentLevelName }}</span>
                      </span>
                    </div>
                    <p class="apple-user-meta">{{ userMetaText }}</p>
                  </div>
                </div>

                <div class="apple-user-actions-grid">
                  <router-link
                    to="/me/orders"
                    class="apple-user-action-tile apple-user-action-tile-primary"
                    @click="closeUserMenu"
                  >
                    <span class="apple-user-action-head">
                      <span class="apple-user-action-title">{{ t('personalCenter.tabs.orders') }}</span>
                      <span class="apple-user-action-icon" aria-hidden="true">
                        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
                        </svg>
                      </span>
                    </span>
                    <span class="apple-user-action-value-row">
                      <span class="apple-user-action-value">{{ userOrderCountDisplay }}</span>
                    </span>
                  </router-link>

                  <router-link
                    to="/me/wallet"
                    class="apple-user-action-tile apple-user-action-tile-wallet"
                    @click="closeUserMenu"
                  >
                    <span class="apple-user-action-head">
                      <span class="apple-user-action-title">{{ t('personalCenter.tabs.wallet') }}</span>
                      <span class="apple-user-action-icon" aria-hidden="true">
                        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M17 9V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-2m0-6h3v6h-3a3 3 0 110-6z" />
                        </svg>
                      </span>
                    </span>
                    <span class="apple-user-action-value-row">
                      <span class="apple-user-action-value apple-user-action-value-balance">{{ walletAmountDisplay }}</span>
                    </span>
                  </router-link>
                </div>

                <button
                  @click="logoutFromMenu"
                  class="apple-user-logout-row"
                >
                  <span class="apple-user-logout-icon" aria-hidden="true">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
                    </svg>
                  </span>
                  <span>{{ t('navbar.logout') }}</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="container mx-auto px-4 lg:hidden grid grid-cols-[44px_minmax(0,1fr)_44px] items-center gap-3">
      <button
        @click="toggleMobileMenu"
        class="theme-nav-link p-2 min-w-[44px] min-h-[44px] flex items-center justify-center rounded-xl"
        :aria-label="t('navbar.more')"
      >
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.9" d="M4 7h16M4 12h16M4 17h16" />
        </svg>
      </button>

      <router-link to="/" class="apple-brand apple-brand-mobile group relative" :title="brandSiteName">
        <img src="/logo.svg" alt="" class="apple-brand-logo" aria-hidden="true" />
        <span class="apple-brand-copy">
          <span class="apple-brand-name">{{ brandSiteName }}</span>
        </span>
      </router-link>

      <button
        type="button"
        @click="goToMobileOrderEntry"
        class="theme-nav-link mobile-order-entry relative z-10 p-2 min-w-[48px] min-h-[48px] flex items-center justify-center justify-self-end rounded-xl touch-manipulation"
        :aria-label="mobileOrderLabel"
        :title="mobileOrderLabel"
      >
        <svg class="w-5 h-5 shrink-0 opacity-85" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
        </svg>
      </button>
    </div>

  </nav>

  <div class="hidden lg:flex apple-side-toolbar">
    <router-link
      to="/cart"
      class="apple-side-tool-button apple-side-tool-button-cart"
      :aria-label="t('navbar.cart')"
    >
      <svg class="w-4 h-4 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
          d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13l-1.3 2.6a1 1 0 00.9 1.4H19M7 13l.4 2M10 21a1 1 0 100-2 1 1 0 000 2zm8 1a1 1 0 100-2 1 1 0 000 2z" />
      </svg>
      <span
        v-if="cartCount > 0"
        class="theme-nav-badge absolute top-1 right-1"
        :class="{ 'theme-bounce-in': cartBounce }"
      >
        {{ cartCount }}
      </span>
    </router-link>

    <button
      @click="toggleTheme"
      class="apple-side-tool-button"
      :aria-label="theme === 'dark' ? 'Switch to light theme' : 'Switch to dark theme'"
    >
      <SunIcon v-if="theme === 'dark'" class="w-4 h-4" />
      <MoonIcon v-else class="w-4 h-4" />
    </button>

    <div class="lang-switcher apple-side-language-stack" :class="{ 'apple-side-language-stack-open': showLangMenu }">
      <button @click="toggleLangMenu" class="apple-side-tool-button apple-side-tool-button-locale apple-side-language-trigger">
        <span class="apple-side-tool-lang">{{ currentLocale }}</span>
      </button>

      <div v-if="showLangMenu" class="apple-side-language-options">
        <button
          v-for="lang in availableLanguages"
          :key="lang.code"
          @click="changeLanguage(lang.code)"
          class="apple-side-language-option"
          :title="lang.name"
          :aria-label="lang.name"
        >
          {{ lang.code === 'en-US' ? 'EN' : (lang.code === 'zh-CN' ? '简' : '繁') }}
        </button>
      </div>
    </div>
  </div>

  <!-- Teleport drawer outside nav to avoid backdrop-filter containing block bug -->
  <Teleport to="body">
    <!-- Mobile Drawer Overlay -->
    <Transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0"
      enter-to-class="opacity-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100"
      leave-to-class="opacity-0">
      <div v-if="showMobileMenu" class="lg:hidden fixed inset-0 z-[60] bg-black/50" @click="showMobileMenu = false" style="overscroll-behavior: none;"></div>
    </Transition>

    <!-- Mobile Drawer (only items NOT in bottom nav) -->
    <Transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="-translate-x-full"
      enter-to-class="translate-x-0"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="translate-x-0"
      leave-to-class="-translate-x-full">
      <div v-if="showMobileMenu"
        class="lg:hidden fixed left-0 top-0 bottom-0 z-[70] w-72 max-w-[80vw] theme-panel-strong backdrop-blur-xl border-r theme-border overflow-y-auto"
        style="overscroll-behavior: none;">
        <div class="mobile-drawer-shell">
          <div class="mobile-drawer-top">
            <router-link to="/" @click="showMobileMenu = false" class="mobile-drawer-brand-card">
              <img src="/logo.svg" alt="" class="mobile-drawer-brand-logo" aria-hidden="true" />
              <span class="mobile-drawer-brand-copy">
                <span class="mobile-drawer-brand-title">{{ brandSiteName }}</span>
              </span>
            </router-link>

            <div class="mobile-drawer-header-actions">
              <button
                @click="toggleTheme"
                class="mobile-drawer-header-button theme-btn-neutral"
                :aria-label="theme === 'dark' ? 'Switch to light theme' : 'Switch to dark theme'"
              >
                <SunIcon v-if="theme === 'dark'" class="w-4 h-4" />
                <MoonIcon v-else class="w-4 h-4" />
              </button>
              <button @click="showMobileMenu = false" class="mobile-drawer-header-button theme-btn-neutral">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>

          <div class="mobile-drawer-divider" aria-hidden="true"></div>

          <div class="mobile-drawer-quick-wrap">
            <span
              v-if="userAuthStore.isAuthenticated"
              class="mobile-drawer-section-title mobile-drawer-section-title-compact"
            >
              {{ t('navbar.personalCenter') }}
            </span>
            <div class="mobile-drawer-quick grid-cols-2">
              <template v-if="userAuthStore.isAuthenticated">
                <router-link
                  v-for="action in mobilePersonalActions"
                  :key="action.key"
                  :to="action.path"
                  @click="showMobileMenu = false"
                  class="mobile-quick-btn"
                  :class="{
                    'mobile-quick-btn-full': action.wide,
                    'mobile-quick-btn-active': isPathActive(action.path),
                  }"
                >
                  <span class="mobile-quick-btn-icon" aria-hidden="true">
                    <component :is="action.icon" class="w-[1.12rem] h-[1.12rem]" />
                  </span>
                  <span>{{ action.label }}</span>
                </router-link>
              </template>

              <template v-else>
                <router-link
                  to="/guest/orders"
                  @click="showMobileMenu = false"
                  class="mobile-quick-btn"
                >
                  <span class="mobile-quick-btn-icon" aria-hidden="true">
                    <svg class="w-[1.12rem] h-[1.12rem]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
                    </svg>
                  </span>
                  <span>{{ t('navbar.guestOrders') }}</span>
                </router-link>

                <router-link
                  to="/auth/login"
                  @click="showMobileMenu = false"
                  class="mobile-quick-btn"
                >
                  <span class="mobile-quick-btn-icon" aria-hidden="true">
                    <svg class="w-[1.12rem] h-[1.12rem]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" d="M11 16l-4-4m0 0l4-4m-4 4h14m-5 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h7a3 3 0 013 3v1" />
                    </svg>
                  </span>
                  <span>{{ t('navbar.login') }}</span>
                </router-link>
              </template>
            </div>
          </div>

          <section v-if="mobileDrawerItems.length > 0" class="mobile-drawer-section">
            <span class="mobile-drawer-section-title">{{ t('navbar.more') }}</span>
            <div class="mt-2 space-y-1">
              <template v-for="item in mobileDrawerItems" :key="item.key">
                <router-link
                  v-if="item.type === 'route'"
                  :to="item.path"
                  @click="showMobileMenu = false"
                  class="mobile-drawer-nav-item theme-nav-link"
                  active-class="mobile-drawer-nav-item-active theme-nav-link-active"
                >
                  <span class="mobile-drawer-nav-main">
                    <svg class="w-[1.125rem] h-[1.125rem]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" :d="item.icon" />
                    </svg>
                    <span>{{ item.label.startsWith('nav.') ? t(item.label) : item.label }}</span>
                  </span>
                  <span aria-hidden="true">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M9 5l7 7-7 7" />
                    </svg>
                  </span>
                </router-link>
                <a
                  v-else
                  :href="item.path"
                  :target="item.target"
                  rel="noopener noreferrer"
                  @click="showMobileMenu = false"
                  class="mobile-drawer-nav-item theme-nav-link"
                >
                  <span class="mobile-drawer-nav-main">
                    <svg class="w-[1.125rem] h-[1.125rem]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" :d="item.icon" />
                    </svg>
                    <span>{{ item.label }}</span>
                  </span>
                  <span aria-hidden="true">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M9 5l7 7-7 7" />
                    </svg>
                  </span>
                </a>
              </template>
            </div>
          </section>

          <div class="mobile-drawer-footer">
            <section class="mobile-drawer-section">
              <span class="mobile-drawer-section-title">{{ t('navbar.selectLanguage') }}</span>
              <div class="mobile-lang-segment" role="group" :aria-label="t('navbar.selectLanguage')">
                <button
                  v-for="lang in languages"
                  :key="lang.code"
                  @click="changeLanguage(lang.code)"
                  class="mobile-lang-option"
                  :class="{ 'mobile-lang-option-active': appStore.locale === lang.code }"
                  :aria-pressed="appStore.locale === lang.code"
                  :title="lang.name"
                >
                  {{ getLanguageShortCode(lang.code) }}
                </button>
              </div>
            </section>

            <button
              v-if="userAuthStore.isAuthenticated"
              @click="userAuthStore.logout(); showMobileMenu = false"
              class="mobile-logout-btn"
            >
              <svg class="w-[1.125rem] h-[1.125rem]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.75" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
              </svg>
              {{ t('navbar.logout') }}
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { ref, computed, nextTick, onMounted, onUnmounted, watch, type Component, type CSSProperties } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'
import { userOrderAPI, walletAPI } from '../api'
import { useAppStore } from '../stores/app'
import { useCartStore } from '../stores/cart'
import { useUserAuthStore } from '../stores/userAuth'
import { useUserProfileStore } from '../stores/userProfile'
import { getImageUrl } from '../utils/image'
import { useTheme } from '../utils/theme'
import { SunIcon, MoonIcon, ShoppingBagIcon, WalletIcon, MegaphoneIcon, GiftIcon, KeyIcon } from '@heroicons/vue/24/outline'

const { t, locale } = useI18n()
const route = useRoute()
const router = useRouter()
const appStore = useAppStore()
const cartStore = useCartStore()
const userAuthStore = useUserAuthStore()
const userProfileStore = useUserProfileStore()
const { theme, toggleTheme } = useTheme()

const showMobileMenu = ref(false)
const showLangMenu = ref(false)
const showUserMenu = ref(false)
const scrolled = ref(false)
const cartBounce = ref(false)
const desktopNavShell = ref<HTMLElement | null>(null)
const navIndicatorVisible = ref(false)
const navIndicatorStyle = ref<CSSProperties>({
  width: '0px',
  transform: 'translate3d(0, 0, 0)',
})
const hoveredNavKey = ref<string | null>(null)
const userOrderCount = ref<number | null>(null)
const walletBalance = ref<string | null>(null)
const loadingUserSummary = ref(false)
let userMenuCloseTimer: number | null = null

const isListMode = computed(() => appStore.config?.template_mode === 'list')

// 内置导航项定义
const builtinNavDefs: Record<string, { path: string; label: string; icon: string }> = {
  blog: { path: '/blog', label: 'nav.blog', icon: 'M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9a2 2 0 00-2-2h-2' },
  notice: { path: '/notice', label: 'nav.notice', icon: 'M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9' },
  about: { path: '/about', label: 'nav.about', icon: 'M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z' },
}

interface NavItem {
  key: string
  path: string
  label: string
  icon: string
  type: 'route' | 'link'
  target: string
}

interface MobileShortcutAction {
  key: string
  path: string
  label: string
  icon: Component
  wide?: boolean
}

const navConfig = computed(() => appStore.config?.nav_config as {
  builtin?: Record<string, boolean>
  custom_items?: Array<{
    id: number; title: Record<string, string>; link_type: string
    url: string; target: string; sort_order: number; enabled: boolean; icon?: string
  }>
} | undefined)

const getCustomItemTitle = (item: { title?: Record<string, string> }): string => {
  const titles = item.title || {}
  return titles[locale.value] || titles['zh-CN'] || titles['en-US'] || ''
}

const presetIcons: Record<string, string> = {
  link: 'M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1',
  document: 'M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z',
  globe: 'M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9',
  star: 'M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z',
  heart: 'M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z',
  chat: 'M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z',
  gift: 'M12 8v13m0-13V6a4 4 0 00-4-4 4 4 0 00-4 4v2h8zm0 0V6a4 4 0 014-4 4 4 0 014 4v2h-8zM5 8h14a1 1 0 011 1v3H4V9a1 1 0 011-1zm0 4h14v7a2 2 0 01-2 2H7a2 2 0 01-2-2v-7z',
  lightning: 'M13 10V3L4 14h7v7l9-11h-7z',
  shield: 'M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z',
  book: 'M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253',
  code: 'M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4',
  phone: 'M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z',
  map: 'M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z M15 11a3 3 0 11-6 0 3 3 0 016 0z',
  music: 'M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3',
  camera: 'M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z M15 13a3 3 0 11-6 0 3 3 0 016 0z',
}
const defaultIcon = presetIcons.link

const buildCustomNavItems = (): NavItem[] => {
  const items = navConfig.value?.custom_items
  if (!Array.isArray(items)) return []
  return items
    .filter((item) => item.enabled)
    .sort((a, b) => (a.sort_order || 0) - (b.sort_order || 0))
    .map((item) => {
      const icon = (presetIcons[item.icon as string] || defaultIcon) as string
      return {
        key: `custom-${item.id}`,
        path: item.url || '',
        label: getCustomItemTitle(item),
        icon,
        type: item.link_type === 'external' ? 'link' as const : 'route' as const,
        target: item.target || '_self',
      }
    })
    .filter((item) => item.label && item.path)
}

const buildBuiltinNavItems = (): NavItem[] => {
  const builtin = navConfig.value?.builtin
  const result: NavItem[] = []
  for (const [key, def] of Object.entries(builtinNavDefs)) {
    if (builtin && builtin[key] === false) continue
    result.push({ key, path: def.path, label: def.label, icon: def.icon, type: 'route', target: '_self' })
  }
  return result
}

const menuItems = computed<NavItem[]>(() => {
  const items: NavItem[] = [
    { key: 'home', path: '/', label: 'nav.home', icon: 'M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-4 0a1 1 0 01-1-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 01-1 1', type: 'route', target: '_self' },
  ]
  if (!isListMode.value) {
    items.push({ key: 'products', path: '/products', label: 'nav.products', icon: 'M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z', type: 'route', target: '_self' })
  }
  items.push(...buildBuiltinNavItems())
  items.push(...buildCustomNavItems())
  return items
})

// Mobile drawer only shows items NOT in the bottom nav (Home, Products, Cart, Me are in bottom nav)
const mobileDrawerItems = computed<NavItem[]>(() => {
  const items: NavItem[] = [...buildBuiltinNavItems(), ...buildCustomNavItems()]
  return items
})

const mobilePersonalActions = computed<MobileShortcutAction[]>(() => [
  {
    key: 'orders',
    path: '/me/orders',
    label: t('personalCenter.tabs.orders'),
    icon: ShoppingBagIcon,
  },
  {
    key: 'wallet',
    path: '/me/wallet',
    label: t('personalCenter.tabs.wallet'),
    icon: WalletIcon,
  },
  {
    key: 'affiliate',
    path: '/me/affiliate',
    label: t('personalCenter.tabs.affiliate'),
    icon: MegaphoneIcon,
  },
  {
    key: 'gift-card',
    path: '/me/gift-cards',
    label: t('personalCenter.tabs.giftCard'),
    icon: GiftIcon,
  },
  {
    key: 'api',
    path: '/me/api',
    label: t('personalCenter.tabs.api'),
    icon: KeyIcon,
    wide: true,
  },
])

const languages = [
  { code: 'zh-CN', name: '简体中文' },
  { code: 'zh-TW', name: '繁體中文' },
  { code: 'en-US', name: 'English' },
]

const currentLocale = computed(() => {
  const lang = languages.find(l => l.code === appStore.locale)
  if (!lang) return 'CN'
  return lang.code === 'en-US' ? 'EN' : (lang.code === 'zh-CN' ? '简' : '繁')
})

const getLanguageShortCode = (langCode: string) => (
  langCode === 'en-US' ? 'EN' : (langCode === 'zh-CN' ? '简' : '繁')
)

const availableLanguages = computed(() => languages.filter((lang) => lang.code !== appStore.locale))

const cartCount = computed(() => cartStore.totalItems)

const mobileOrderPath = computed(() => userAuthStore.isAuthenticated ? '/me/orders' : '/guest/orders')

const mobileOrderLabel = computed(() => userAuthStore.isAuthenticated ? t('personalCenter.tabs.orders') : t('navbar.guestOrders'))

const goToMobileOrderEntry = async () => {
  const target = mobileOrderPath.value
  if (route.path === target) {
    window.scrollTo({ top: 0, behavior: 'smooth' })
    return
  }
  await router.push(target)
}

const brandSiteName = computed(() => {
  const text = String(appStore.config?.brand?.site_name || '').trim()
  return text !== '' ? text : 'Dujiao-Next'
})

const userDisplayName = computed(() => {
  const nickname = String(userAuthStore.user?.nickname || '').trim()
  const email = String(userAuthStore.user?.email || '').trim()
  if (nickname !== '') return nickname
  if (email !== '') return email.split('@')[0] || email
  return t('navbar.personalCenter')
})

const userMetaText = computed(() => {
  const email = String(userAuthStore.user?.email || '').trim()
  return email !== '' ? email : brandSiteName.value
})

const userInitial = computed(() => {
  const raw = userDisplayName.value.trim()
  return raw !== '' ? raw.charAt(0).toUpperCase() : 'U'
})

const currentUserLevel = computed(() => {
  const levelId = Number(userAuthStore.user?.member_level_id || 0)
  if (!levelId || userProfileStore.memberLevels.length === 0) return null
  return userProfileStore.memberLevels.find((level) => level.id === levelId) || null
})

const isImagePath = (icon: string | undefined | null) => Boolean(icon?.startsWith('/uploads/') || icon?.startsWith('http'))

const currentLevelName = computed(() => {
  const level = currentUserLevel.value
  if (!level) return t('personalCenter.memberLevel.defaultLevel')
  const loc = locale.value as string
  return level.name[loc] || level.name['zh-CN'] || level.name['en'] || level.slug || t('personalCenter.memberLevel.defaultLevel')
})

const currentLevelIcon = computed(() => String(currentUserLevel.value?.icon || '').trim())

const ensureMemberLevelsLoaded = async () => {
  if (!userAuthStore.isAuthenticated || userProfileStore.memberLevels.length > 0) return
  await userProfileStore.loadMemberLevels()
}

const walletAmountDisplay = computed(() => {
  if (loadingUserSummary.value && walletBalance.value === null) return t('common.loading')
  const value = String(walletBalance.value ?? '').trim()
  return value !== '' ? value : '0.00'
})

const userOrderCountDisplay = computed(() => {
  if (loadingUserSummary.value && userOrderCount.value === null) return t('common.loading')
  return String(userOrderCount.value ?? 0)
})

const resetUserSummary = () => {
  userOrderCount.value = null
  walletBalance.value = null
  loadingUserSummary.value = false
}

const loadUserSummary = async () => {
  if (!userAuthStore.isAuthenticated || loadingUserSummary.value) return
  loadingUserSummary.value = true
  try {
    const [orderResult, walletResult] = await Promise.allSettled([
      userOrderAPI.list({ page: 1, page_size: 1 }),
      walletAPI.account(),
    ])

    if (orderResult.status === 'fulfilled') {
      const pagination = orderResult.value.data.pagination
      const data = orderResult.value.data.data
      userOrderCount.value = Number(pagination?.total ?? (Array.isArray(data) ? data.length : 0))
    } else if (userOrderCount.value === null) {
      userOrderCount.value = 0
    }

    if (walletResult.status === 'fulfilled') {
      walletBalance.value = String(walletResult.value.data.data?.balance ?? '0.00')
    } else if (walletBalance.value === null) {
      walletBalance.value = '0.00'
    }
  } finally {
    loadingUserSummary.value = false
  }
}

const isRouteItemActive = (item: NavItem): boolean => {
  if (item.type !== 'route') return false
  if (item.path === '/') return route.path === '/'
  return route.path === item.path || route.path.startsWith(`${item.path}/`)
}

const isPathActive = (path: string): boolean => route.path === path || route.path.startsWith(`${path}/`)

const syncNavIndicator = (preferredKey?: string | null) => {
  const shell = desktopNavShell.value
  if (!shell) return

  const fallbackKey = menuItems.value.find(isRouteItemActive)?.key ?? null
  const key = preferredKey ?? hoveredNavKey.value ?? fallbackKey

  if (!key) {
    navIndicatorVisible.value = false
    return
  }

  const target = shell.querySelector<HTMLElement>(`[data-nav-key="${key}"]`)
  if (!target) {
    navIndicatorVisible.value = false
    return
  }

  const shellRect = shell.getBoundingClientRect()
  const targetRect = target.getBoundingClientRect()
  navIndicatorStyle.value = {
    width: `${targetRect.width}px`,
    transform: `translate3d(${targetRect.left - shellRect.left}px, 0, 0)`,
  }
  navIndicatorVisible.value = true
}

const setNavHover = (key: string) => {
  hoveredNavKey.value = key
  syncNavIndicator(key)
}

const clearNavHover = () => {
  hoveredNavKey.value = null
  syncNavIndicator()
}

const handleDesktopNavFocusOut = (event: FocusEvent) => {
  const relatedTarget = event.relatedTarget as HTMLElement | null
  if (!relatedTarget || !desktopNavShell.value?.contains(relatedTarget)) {
    clearNavHover()
  }
}

const toggleMobileMenu = () => {
  showMobileMenu.value = !showMobileMenu.value
  if (showMobileMenu.value) {
    showLangMenu.value = false
    showUserMenu.value = false
  }
}

const toggleLangMenu = () => {
  if (!showLangMenu.value) showUserMenu.value = false
  showLangMenu.value = !showLangMenu.value
}

const clearUserMenuCloseTimer = () => {
  if (userMenuCloseTimer !== null) {
    window.clearTimeout(userMenuCloseTimer)
    userMenuCloseTimer = null
  }
}

const openUserMenu = () => {
  clearUserMenuCloseTimer()
  showLangMenu.value = false
  showUserMenu.value = true
  void ensureMemberLevelsLoaded()
  void loadUserSummary()
}

const closeUserMenu = () => {
  clearUserMenuCloseTimer()
  showUserMenu.value = false
}

const scheduleCloseUserMenu = () => {
  clearUserMenuCloseTimer()
  userMenuCloseTimer = window.setTimeout(() => {
    showUserMenu.value = false
    userMenuCloseTimer = null
  }, 120)
}

const handleUserMenuFocusOut = (event: FocusEvent) => {
  const relatedTarget = event.relatedTarget as HTMLElement | null
  if (!relatedTarget || !relatedTarget.closest('.user-menu')) {
    closeUserMenu()
  }
}

const changeLanguage = (langCode: string) => {
  appStore.setLocale(langCode)
  locale.value = langCode
  showLangMenu.value = false
}

const logoutFromMenu = () => {
  closeUserMenu()
  userAuthStore.logout()
}

const handleScroll = () => {
  scrolled.value = window.scrollY > 20
}

const handleResize = () => {
  syncNavIndicator()
}

// Click outside to close menus
const handleClickOutside = (event: MouseEvent) => {
  const target = event.target as HTMLElement
  if (!target.closest('.lang-switcher')) {
    showLangMenu.value = false
  }
  if (!target.closest('.user-menu')) {
    showUserMenu.value = false
  }
}

// Cart badge bounce animation on count change
watch(cartCount, (newVal, oldVal) => {
  if (newVal > oldVal) {
    cartBounce.value = true
    setTimeout(() => { cartBounce.value = false }, 400)
  }
})

watch(
  () => userAuthStore.isAuthenticated,
  (authenticated) => {
    if (authenticated) {
      void ensureMemberLevelsLoaded()
      void loadUserSummary()
    } else {
      resetUserSummary()
      showUserMenu.value = false
    }
  },
  { immediate: true }
)

watch(
  [menuItems, () => route.path, () => locale.value],
  () => {
    nextTick(() => syncNavIndicator())
  },
  { immediate: true }
)

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  window.addEventListener('scroll', handleScroll, { passive: true })
  window.addEventListener('resize', handleResize)
  nextTick(() => syncNavIndicator())
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('resize', handleResize)
  clearUserMenuCloseTimer()
})
</script>

<style scoped>
.scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
.scrollbar-hide::-webkit-scrollbar {
  display: none;
}

.apple-brand {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  min-height: 2.5rem;
  max-width: min(62vw, 18rem);
  padding: 0;
  transition: transform 0.18s ease, opacity 0.18s ease;
}

.apple-brand:hover {
  transform: translateY(-0.5px);
  opacity: 0.98;
}

.apple-brand:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
  border-radius: 1rem;
}

.apple-brand-logo {
  display: block;
  width: 1.92rem;
  height: 1.92rem;
  flex-shrink: 0;
  object-fit: contain;
  transition: transform 0.18s ease, opacity 0.18s ease;
}

.apple-brand:hover .apple-brand-logo {
  transform: translateY(-0.5px) scale(1.01);
  opacity: 0.96;
}

.apple-brand-copy {
  min-width: 0;
  display: flex;
  align-items: center;
}

.apple-brand-mobile {
  justify-self: center;
  min-width: 0;
  max-width: min(100%, 13rem);
}

.apple-brand-name {
  --apple-brand-name-gradient: linear-gradient(
    180deg,
    color-mix(in oklab, white 70%, var(--ui-text-primary) 30%) 0%,
    color-mix(in oklab, white 36%, var(--ui-text-primary) 64%) 22%,
    color-mix(in oklab, var(--ui-text-primary) 86%, white 14%) 54%,
    color-mix(in oklab, var(--ui-text-primary) 94%, black 6%) 100%
  );
  --apple-brand-name-shadow: 0 1px 0 rgba(255, 255, 255, 0.12);
  --apple-brand-name-stroke: 0.2px color-mix(in oklab, var(--ui-text-primary) 72%, black 28%);
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: var(--ui-text-primary);
  font-size: clamp(1.02rem, 0.92rem + 0.28vw, 1.18rem);
  font-weight: 800;
  line-height: 1.02;
  letter-spacing: -0.03em;
  background-image: var(--apple-brand-name-gradient);
  text-shadow: var(--apple-brand-name-shadow);
  -webkit-text-stroke: var(--apple-brand-name-stroke);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}

:global(.dark) .apple-brand-name {
  --apple-brand-name-gradient: linear-gradient(
    180deg,
    color-mix(in oklab, white 94%, var(--ui-text-primary) 6%) 0%,
    color-mix(in oklab, white 76%, var(--ui-text-primary) 24%) 20%,
    color-mix(in oklab, white 52%, var(--ui-text-primary) 48%) 52%,
    color-mix(in oklab, var(--ui-text-primary) 82%, white 18%) 100%
  );
  --apple-brand-name-shadow: 0 1px 0 rgba(255, 255, 255, 0.08), 0 8px 18px rgba(0, 0, 0, 0.18);
  --apple-brand-name-stroke: 0.2px color-mix(in oklab, white 34%, var(--ui-text-primary) 66%);
}

.apple-nav-shell {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 0.2rem;
  max-width: min(100%, 54rem);
  padding: 0.32rem;
  border-radius: 999px;
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  background: color-mix(in oklab, var(--ui-bg-overlay-strong) 90%, transparent);
  box-shadow: var(--ui-shadow-1);
  overflow: hidden;
}

.apple-nav-indicator {
  position: absolute;
  top: 0.32rem;
  bottom: 0.32rem;
  left: 0;
  border-radius: 999px;
  background:
    linear-gradient(180deg, color-mix(in oklab, var(--ui-bg-elevated) 88%, white 12%), color-mix(in oklab, var(--ui-accent-soft) 38%, transparent)),
    color-mix(in oklab, var(--ui-bg-elevated) 92%, transparent);
  box-shadow:
    inset 0 0 0 1px color-mix(in oklab, var(--ui-accent) 16%, var(--ui-border)),
    0 14px 24px -22px color-mix(in oklab, var(--ui-accent) 44%, transparent);
  opacity: 0;
  pointer-events: none;
  transition:
    transform 0.38s cubic-bezier(0.22, 1, 0.36, 1),
    width 0.38s cubic-bezier(0.22, 1, 0.36, 1),
    opacity 0.2s ease;
}

.apple-nav-indicator-visible {
  opacity: 1;
}

.apple-nav-link {
  position: relative;
  z-index: 1;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.45rem;
  min-height: 2.45rem;
  padding: 0 1rem;
  border-radius: 999px;
  color: var(--ui-text-muted);
  font-size: 0.82rem;
  font-weight: 500;
  line-height: 1;
  letter-spacing: -0.01em;
  white-space: nowrap;
  transition: color 0.2s ease, background-color 0.2s ease, box-shadow 0.2s ease;
}

.apple-nav-link-icon {
  width: 0.95rem;
  height: 0.95rem;
  flex-shrink: 0;
  opacity: 0.72;
}

.apple-nav-link:hover {
  color: var(--ui-text-primary);
}

.apple-nav-link:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
}

.apple-nav-link-active {
  color: var(--ui-text-primary) !important;
}

.apple-nav-link:hover .apple-nav-link-icon,
.apple-nav-link:focus-visible .apple-nav-link-icon,
.apple-nav-link-active .apple-nav-link-icon {
  opacity: 0.95;
}

.apple-action-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.45rem;
  min-height: 2.5rem;
  padding: 0 1rem;
  border-radius: 999px;
  border: 1px solid transparent;
  font-size: 0.81rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  white-space: nowrap;
  transition: transform 0.2s ease, color 0.2s ease, background-color 0.2s ease, border-color 0.2s ease, box-shadow 0.2s ease;
}

.apple-action-icon {
  width: 2.5rem;
  padding: 0;
}

.apple-action-pill:hover {
  transform: translateY(-1px);
}

.apple-action-pill:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
}

.apple-action-pill-secondary {
  color: var(--ui-text-secondary);
  background: color-mix(in oklab, var(--ui-bg-soft) 88%, transparent);
  border-color: var(--ui-border);
}

.apple-action-pill-secondary:hover {
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 84%, transparent);
}

.apple-action-pill-primary {
  color: var(--ui-text-on-accent);
  background: linear-gradient(180deg, color-mix(in oklab, var(--ui-accent) 88%, white 12%), var(--ui-accent));
  border-color: color-mix(in oklab, var(--ui-accent) 72%, transparent);
  box-shadow: 0 14px 24px -18px color-mix(in oklab, var(--ui-accent) 58%, transparent);
}

.apple-action-pill-primary:hover {
  background: linear-gradient(180deg, color-mix(in oklab, var(--ui-accent-hover) 86%, white 14%), var(--ui-accent-hover));
  border-color: color-mix(in oklab, var(--ui-accent-hover) 70%, transparent);
}

.apple-user-menu {
  position: relative;
  display: inline-flex;
  align-items: center;
}

.apple-user-menu-panel {
  --apple-user-menu-bg:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 94%, white 6%),
      color-mix(in oklab, var(--ui-bg-soft) 82%, var(--ui-bg-elevated) 18%)
    ),
    color-mix(in oklab, var(--ui-bg-elevated) 98%, transparent);
  --apple-user-menu-border: color-mix(in oklab, var(--ui-border-strong) 46%, var(--ui-border));
  --apple-user-menu-text-primary: var(--ui-text-primary);
  --apple-user-menu-text-secondary: var(--ui-text-secondary);
  --apple-user-level-bg: linear-gradient(135deg, rgba(123, 113, 255, 0.96), rgba(94, 88, 229, 0.92));
  --apple-user-level-color: rgba(255, 255, 255, 0.98);
  --apple-user-level-shadow: 0 12px 22px -18px rgba(94, 88, 229, 0.52);
  --apple-user-orders-bg:
    linear-gradient(
      180deg,
      color-mix(in oklab, #4f7cff 14%, var(--ui-bg-elevated)),
      color-mix(in oklab, #4f7cff 8%, var(--ui-bg-soft))
    );
  --apple-user-orders-border: color-mix(in oklab, #4f7cff 24%, var(--ui-border));
  --apple-user-orders-color: #3e63f4;
  --apple-user-wallet-bg:
    linear-gradient(
      180deg,
      color-mix(in oklab, #f7b534 18%, var(--ui-bg-elevated)),
      color-mix(in oklab, #f7b534 10%, var(--ui-bg-soft))
    );
  --apple-user-wallet-border: color-mix(in oklab, #f7b534 24%, var(--ui-border));
  --apple-user-wallet-color: #d28600;
  --apple-user-logout-bg:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-danger-soft) 92%, var(--ui-bg-elevated)),
      color-mix(in oklab, var(--ui-danger-soft) 74%, var(--ui-bg-soft))
    );
  --apple-user-logout-border: color-mix(in oklab, var(--ui-danger) 26%, var(--ui-border));
  --apple-user-logout-color: var(--ui-danger);
  min-width: 18.5rem;
  margin-top: 0.6rem;
  padding: 0.5rem;
  border-radius: 1.5rem;
  border-color: var(--apple-user-menu-border);
  background: var(--apple-user-menu-bg);
  box-shadow: 0 30px 60px -32px color-mix(in oklab, black 30%, transparent);
  animation: apple-user-menu-pop 180ms var(--ui-ease-out);
  transform-origin: top right;
}

:global(.dark) .apple-user-menu-panel {
  --apple-user-menu-bg:
    linear-gradient(180deg, rgba(60, 60, 64, 0.96), rgba(37, 37, 41, 0.98)),
    rgba(30, 30, 33, 0.98);
  --apple-user-menu-border: rgba(255, 255, 255, 0.08);
  --apple-user-menu-text-primary: rgba(255, 255, 255, 0.96);
  --apple-user-menu-text-secondary: rgba(255, 255, 255, 0.58);
  --apple-user-level-shadow: 0 10px 18px -14px rgba(132, 122, 255, 0.8);
  --apple-user-orders-bg: linear-gradient(180deg, rgba(45, 59, 95, 0.98), rgba(44, 58, 94, 0.9));
  --apple-user-orders-border: rgba(96, 130, 255, 0.18);
  --apple-user-orders-color: #8bb0ff;
  --apple-user-wallet-bg: linear-gradient(180deg, rgba(66, 72, 46, 0.98), rgba(54, 59, 39, 0.92));
  --apple-user-wallet-border: rgba(255, 196, 94, 0.16);
  --apple-user-wallet-color: #ffbf47;
  --apple-user-logout-bg: linear-gradient(180deg, rgba(87, 46, 58, 0.72), rgba(74, 39, 49, 0.62));
  --apple-user-logout-border: rgba(255, 106, 136, 0.14);
  --apple-user-logout-color: #ff6a88;
}

.apple-user-card {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.apple-user-card-head {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 0.2rem 0.15rem 0;
}

.apple-user-avatar {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 3rem;
  height: 3rem;
  flex-shrink: 0;
  border-radius: 999px;
  background:
    radial-gradient(circle at 30% 28%, rgba(255, 255, 255, 0.36), transparent 38%),
    linear-gradient(135deg, #4f7cff, #7b56ff 58%, #5f7cff);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.34), 0 16px 24px -18px rgba(79, 124, 255, 0.75);
  color: #ffffff;
  font-size: 1rem;
  font-weight: 800;
  letter-spacing: -0.02em;
}

.apple-user-name-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-width: 0;
}

.apple-user-name {
  display: block;
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: var(--apple-user-menu-text-primary);
  font-size: 1.05rem;
  font-weight: 800;
  letter-spacing: -0.03em;
}

.apple-user-level {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.38rem;
  min-height: 1.52rem;
  padding: 0.18rem 0.62rem 0.18rem 0.48rem;
  border-radius: 999px;
  background: var(--apple-user-level-bg);
  color: var(--apple-user-level-color);
  font-size: 0.68rem;
  font-weight: 800;
  letter-spacing: 0.02em;
  box-shadow: var(--apple-user-level-shadow);
}

.apple-user-level-icon-image {
  width: 0.86rem;
  height: 0.86rem;
  flex-shrink: 0;
  object-fit: contain;
}

.apple-user-level-icon-text {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 0.78rem;
  line-height: 1;
}

.apple-user-meta {
  margin-top: 0.18rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: var(--apple-user-menu-text-secondary);
  font-size: 0.78rem;
  font-weight: 500;
}

.apple-user-actions-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.65rem;
}

.apple-user-action-tile {
  position: relative;
  display: flex;
  min-height: 4.9rem;
  flex-direction: column;
  justify-content: space-between;
  gap: 0.72rem;
  overflow: hidden;
  padding: 0.92rem 0.95rem;
  border-radius: 1.15rem;
  border: 1px solid transparent;
  text-align: left;
  transition: transform 0.18s ease, border-color 0.18s ease, box-shadow 0.18s ease;
}

.apple-user-action-tile::before {
  content: '';
  position: absolute;
  inset: 0;
  background:
    radial-gradient(circle at 100% 0%, color-mix(in oklab, currentColor 12%, transparent), transparent 40%),
    linear-gradient(180deg, rgba(255, 255, 255, 0.08), transparent 34%);
  opacity: 0.9;
  pointer-events: none;
}

.apple-user-action-tile:hover {
  transform: translateY(-1px);
}

.apple-user-action-tile:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px rgba(10, 132, 255, 0.24);
}

.apple-user-action-head {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  min-width: 0;
}

.apple-user-action-title {
  color: currentColor;
  min-width: 0;
  opacity: 0.76;
  font-size: 0.76rem;
  font-weight: 700;
  letter-spacing: -0.01em;
}

.apple-user-action-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 1.9rem;
  height: 1.9rem;
  flex-shrink: 0;
  border-radius: 0.72rem;
  border: 1px solid color-mix(in oklab, currentColor 18%, transparent);
  background:
    linear-gradient(180deg, color-mix(in oklab, currentColor 12%, transparent), color-mix(in oklab, currentColor 6%, transparent)),
    color-mix(in oklab, currentColor 7%, transparent);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.12);
}

.apple-user-action-value-row {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: flex-end;
  min-width: 0;
}

.apple-user-action-value {
  color: currentColor;
  font-size: 1.86rem;
  line-height: 0.94;
  font-weight: 800;
  letter-spacing: -0.05em;
  font-variant-numeric: tabular-nums;
}

.apple-user-action-value-balance {
  font-size: 1.3rem;
  letter-spacing: -0.035em;
}

.apple-user-action-tile-primary {
  background: var(--apple-user-orders-bg);
  border-color: var(--apple-user-orders-border);
  color: var(--apple-user-orders-color);
  box-shadow: 0 18px 28px -24px color-mix(in oklab, var(--apple-user-orders-color) 32%, transparent);
}

.apple-user-action-tile-primary:hover {
  border-color: color-mix(in oklab, var(--apple-user-orders-color) 32%, var(--apple-user-orders-border));
}

.apple-user-action-tile-wallet {
  background: var(--apple-user-wallet-bg);
  border-color: var(--apple-user-wallet-border);
  color: var(--apple-user-wallet-color);
  box-shadow: 0 18px 28px -24px color-mix(in oklab, var(--apple-user-wallet-color) 28%, transparent);
}

.apple-user-action-tile-wallet:hover {
  border-color: color-mix(in oklab, var(--apple-user-wallet-color) 30%, var(--apple-user-wallet-border));
}

.apple-user-logout-row {
  display: inline-flex;
  width: 100%;
  align-items: center;
  justify-content: center;
  gap: 0.55rem;
  min-height: 2.95rem;
  border-radius: 1rem;
  border: 1px solid var(--apple-user-logout-border);
  background: var(--apple-user-logout-bg);
  color: var(--apple-user-logout-color);
  font-size: 0.9rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  transition: transform 0.18s ease, border-color 0.18s ease, background-color 0.18s ease, box-shadow 0.18s ease;
}

.apple-user-logout-row:hover {
  transform: translateY(-1px);
  border-color: color-mix(in oklab, var(--apple-user-logout-color) 30%, var(--apple-user-logout-border));
  box-shadow: 0 16px 24px -22px color-mix(in oklab, var(--apple-user-logout-color) 28%, transparent);
}

.apple-user-logout-row:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px color-mix(in oklab, var(--apple-user-logout-color) 20%, transparent);
}

.apple-user-logout-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 1.9rem;
  height: 1.9rem;
  border-radius: 999px;
  background: color-mix(in oklab, var(--apple-user-logout-color) 12%, transparent);
}

@keyframes apple-user-menu-pop {
  from {
    opacity: 0;
    transform: translate3d(0, -8px, 0) scale(0.96);
  }
  to {
    opacity: 1;
    transform: translate3d(0, 0, 0) scale(1);
  }
}

.apple-side-toolbar {
  position: fixed;
  top: 50%;
  right: clamp(0.85rem, 2vw, 1.35rem);
  z-index: 40;
  flex-direction: column;
  gap: 0.48rem;
  transform: translateY(-50%);
  pointer-events: none;
}

.apple-side-tool-button {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.8rem;
  height: 2.8rem;
  border-radius: 0.78rem;
  pointer-events: auto;
  color: var(--ui-text-muted);
  background:
    linear-gradient(180deg, color-mix(in oklab, var(--ui-bg-elevated) 78%, white 22%), color-mix(in oklab, var(--ui-bg-soft) 80%, transparent)),
    color-mix(in oklab, var(--ui-bg-elevated) 88%, transparent);
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 18px 28px -22px color-mix(in oklab, black 24%, transparent);
  transition: transform 0.2s ease, color 0.2s ease, background-color 0.2s ease, border-color 0.2s ease, box-shadow 0.2s ease;
}

.apple-side-tool-button:hover {
  transform: translateY(-1px);
  color: var(--ui-text-primary);
  border-color: color-mix(in oklab, var(--ui-border-strong) 78%, white 22%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.24),
    0 20px 30px -22px color-mix(in oklab, black 28%, transparent);
}

.apple-side-tool-button:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
}

.apple-side-tool-button-cart {
  overflow: visible;
}

.apple-side-language-stack {
  display: flex;
  width: 2.8rem;
  flex-direction: column;
  overflow: hidden;
  border-radius: 0.78rem;
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  background:
    linear-gradient(180deg, color-mix(in oklab, var(--ui-bg-elevated) 78%, white 22%), color-mix(in oklab, var(--ui-bg-soft) 80%, transparent)),
    color-mix(in oklab, var(--ui-bg-elevated) 88%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 18px 28px -22px color-mix(in oklab, black 24%, transparent);
  pointer-events: auto;
  transition: transform 0.2s ease, border-color 0.2s ease, box-shadow 0.2s ease;
}

.apple-side-language-stack:hover,
.apple-side-language-stack-open {
  transform: translateY(-1px);
  border-color: color-mix(in oklab, var(--ui-border-strong) 78%, white 22%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.24),
    0 20px 30px -22px color-mix(in oklab, black 28%, transparent);
}

.apple-side-language-trigger {
  width: 100%;
  border: 0;
  border-radius: 0;
  background: transparent;
  box-shadow: none;
}

.apple-side-language-trigger:hover,
.apple-side-language-trigger:focus-visible {
  transform: none;
  border-color: transparent;
}

.apple-side-language-trigger:focus-visible {
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
}

.apple-side-tool-button-locale {
  gap: 0;
}

.apple-side-tool-lang {
  font-size: 0.74rem;
  font-weight: 800;
  letter-spacing: 0.12em;
  line-height: 1;
  text-transform: uppercase;
}

.apple-side-language-options {
  display: flex;
  flex-direction: column;
  border-top: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  animation: apple-user-menu-pop 180ms var(--ui-ease-out);
}

.apple-side-language-option {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  min-height: 2.25rem;
  color: var(--ui-text-secondary);
  font-size: 0.74rem;
  font-weight: 800;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  transition: color 0.18s ease, background-color 0.18s ease;
}

.apple-side-language-option + .apple-side-language-option {
  border-top: 1px solid color-mix(in oklab, var(--ui-border) 76%, transparent);
}

.apple-side-language-option:hover {
  color: var(--ui-text-primary);
  background: color-mix(in oklab, var(--ui-bg-soft) 84%, transparent);
}

.mobile-drawer-shell {
  min-height: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;
}

.mobile-drawer-footer {
  display: flex;
  flex-direction: column;
  gap: 0.95rem;
  margin-top: auto;
  padding-top: 0.2rem;
}

.mobile-drawer-divider {
  height: 1px;
  width: 100%;
  background: color-mix(in oklab, var(--ui-border) 92%, transparent);
}

.mobile-drawer-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.7rem;
}

.mobile-drawer-brand-card {
  display: flex;
  min-width: 0;
  flex: 1;
  align-items: center;
  gap: 0.5rem;
  padding: 0.1rem 0;
}

.mobile-drawer-brand-logo {
  display: block;
  width: 2.08rem;
  height: 2.08rem;
  flex-shrink: 0;
  object-fit: contain;
}

.mobile-drawer-brand-copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  justify-content: center;
}

.mobile-drawer-brand-title {
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: var(--ui-text-primary);
  font-size: 1rem;
  font-weight: 900;
  letter-spacing: -0.03em;
}

.mobile-drawer-header-actions {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.mobile-drawer-header-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  border-radius: 999px;
}

.mobile-drawer-header-button.theme-btn-neutral {
  border: 0;
  background: transparent;
  box-shadow: none;
  color: var(--ui-text-secondary);
}

.mobile-drawer-header-button.theme-btn-neutral:hover {
  background: color-mix(in oklab, var(--ui-bg-soft) 82%, transparent);
  color: var(--ui-text-primary);
}

.mobile-order-entry {
  appearance: none;
  border-color: transparent !important;
  background: transparent !important;
  box-shadow: none !important;
}

.mobile-order-entry:hover,
.mobile-order-entry:focus-visible,
.mobile-order-entry:active {
  border-color: transparent !important;
  background: transparent !important;
  box-shadow: none !important;
}

.mobile-drawer-quick-wrap {
  display: grid;
  gap: 0.5rem;
}

.mobile-drawer-quick {
  display: grid;
  gap: 0.55rem;
}

.mobile-drawer-section {
  padding-top: 0.25rem;
}

.mobile-drawer-section-title {
  display: inline-flex;
  align-items: center;
  padding: 0 0.2rem;
  color: var(--ui-text-muted);
  font-size: 0.68rem;
  font-weight: 800;
  letter-spacing: 0.14em;
  line-height: 1;
  text-transform: uppercase;
}

.mobile-drawer-section-title-compact {
  padding-left: 0.05rem;
}

.mobile-quick-btn {
  min-height: 2.7rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.45rem;
  padding: 0.35rem 0.45rem;
  border-radius: 0.8rem;
  border: 1px solid color-mix(in oklab, var(--ui-border) 72%, transparent);
  background: color-mix(in oklab, var(--ui-bg-elevated) 86%, transparent);
  color: var(--ui-text-secondary);
  text-align: center;
  line-height: 1.15;
  font-size: 0.78rem;
  font-weight: 700;
  transition: border-color 180ms ease, background-color 180ms ease, color 180ms ease, transform 180ms ease;
}

.mobile-quick-btn:hover {
  color: var(--ui-text-primary);
  transform: translateY(-1px);
}

.mobile-quick-btn:active {
  transform: scale(0.98);
}

.mobile-quick-btn-active {
  border-color: color-mix(in oklab, var(--ui-accent) 35%, var(--ui-border));
  background: color-mix(in oklab, var(--ui-accent-soft) 48%, var(--ui-bg-elevated));
  color: var(--ui-text-primary);
}

.mobile-quick-btn-full {
  grid-column: 1 / -1;
}

.mobile-quick-btn-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.mobile-drawer-nav-item {
  width: 100%;
  min-height: 2.95rem;
  display: inline-flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.6rem;
  padding: 0.7rem 0.8rem;
  border-radius: 0.8rem;
  font-size: 0.88rem;
  font-weight: 600;
  opacity: 0;
  transform: translateY(8px);
  animation: mobile-drawer-item-in 380ms cubic-bezier(0.22, 1, 0.36, 1) forwards;
}

.mobile-drawer-nav-item-active {
  opacity: 1;
  transform: translateY(0);
}

.mobile-drawer-nav-main {
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
  min-width: 0;
}

.mobile-drawer-nav-main span {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.mobile-drawer-cta {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  width: 100%;
  min-height: 3.35rem;
  padding: 0.82rem 0.92rem;
  border-radius: 1.08rem;
  border: 1px solid transparent;
  text-align: left;
}

.mobile-drawer-cta-primary {
  background: color-mix(in oklab, var(--ui-accent-soft) 70%, var(--ui-bg-elevated));
  border-color: color-mix(in oklab, var(--ui-accent) 24%, transparent);
  color: var(--ui-text-primary);
}

.mobile-drawer-cta-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.2rem;
  height: 2.2rem;
  flex-shrink: 0;
  border-radius: 0.9rem;
  background: color-mix(in oklab, currentColor 10%, transparent);
}

.mobile-drawer-cta-copy {
  display: flex;
  min-width: 0;
  flex: 1;
  flex-direction: column;
  gap: 0.14rem;
}

.mobile-drawer-cta-title {
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-size: 0.9rem;
  font-weight: 800;
  letter-spacing: -0.02em;
}

.mobile-drawer-cta-subtitle {
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  opacity: 0.72;
  font-size: 0.72rem;
  font-weight: 600;
}

.mobile-lang-segment {
  margin-top: 0.55rem;
  padding: 0.2rem;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.25rem;
  border-radius: 999px;
  border: 1px solid color-mix(in oklab, var(--ui-border) 70%, transparent);
  background: color-mix(in oklab, var(--ui-bg-elevated) 84%, transparent);
}

.mobile-lang-option {
  min-height: 2rem;
  border-radius: 999px;
  border: 1px solid transparent;
  color: var(--ui-text-secondary);
  font-size: 0.72rem;
  font-weight: 800;
  letter-spacing: 0.08em;
  transition: border-color 180ms ease, background-color 180ms ease, color 180ms ease, transform 180ms ease;
}

.mobile-lang-option:hover {
  color: var(--ui-text-primary);
}

.mobile-lang-option:active {
  transform: scale(0.97);
}

.mobile-lang-option-active {
  color: var(--ui-accent);
  border-color: color-mix(in oklab, var(--ui-accent) 35%, var(--ui-border));
  background: color-mix(in oklab, var(--ui-accent-soft) 45%, var(--ui-bg-elevated));
}

.mobile-logout-btn {
  width: 100%;
  min-height: 2.85rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.48rem;
  border-radius: 0.8rem;
  border: 1px solid #ef4444;
  background: #ef4444;
  color: #ffffff;
  font-size: 0.82rem;
  font-weight: 700;
  transition: background-color 180ms ease, border-color 180ms ease, box-shadow 180ms ease, transform 180ms ease;
}

.mobile-logout-btn:hover {
  border-color: #dc2626;
  background: #dc2626;
  box-shadow: 0 12px 18px -14px rgba(220, 38, 38, 0.9);
}

.mobile-logout-btn:active {
  transform: scale(0.98);
}

@keyframes mobile-drawer-item-in {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (min-width: 1024px) {
  .mobile-drawer-nav-item {
    animation: none;
    opacity: 1;
    transform: none;
  }
}

.apple-icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.35rem;
  height: 2.35rem;
  border-radius: 999px;
  color: var(--ui-text-muted);
  transition: color 0.2s ease, background-color 0.2s ease, transform 0.2s ease;
}

.apple-icon-button:hover {
  color: var(--ui-text-primary);
  background: color-mix(in oklab, var(--ui-bg-soft) 86%, transparent);
  transform: translateY(-1px);
}

.apple-icon-button:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
}

</style>
