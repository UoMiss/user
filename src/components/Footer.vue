<template>
  <footer
    class="relative theme-panel-strong theme-text-secondary border-t border-gray-200/50 dark:border-white/10 overflow-hidden">
    <div class="container mx-auto px-4 py-10 lg:py-14">
      <div class="grid grid-cols-1 gap-10 lg:grid-cols-4 lg:gap-8">
        <div class="hidden lg:flex flex-col space-y-6 lg:col-span-1 lg:justify-between">
          <div class="space-y-4">
            <router-link to="/" class="footer-brand group" :title="brandSiteName">
              <img
                src="/logo.svg"
                :alt="brandSiteName"
                class="footer-brand-logo"
                loading="lazy"
                decoding="async"
              />
              <h3 class="footer-brand-name">
                {{ brandSiteName }}
              </h3>
            </router-link>
            <p class="max-w-none text-sm leading-relaxed theme-text-muted lg:max-w-xs">
              {{ brandDescription || t('footer.description') }}
            </p>
          </div>

          <div class="hidden border-t border-dashed border-gray-200/60 pt-6 dark:border-white/10 lg:block">
            <p class="text-xs font-medium opacity-50">&copy; {{ currentYear }} {{ brandSiteName }}. {{ t('footer.rights') }}</p>
          </div>
        </div>

        <div :class="['grid gap-8 lg:col-span-2 lg:px-8', footerLinks.length ? 'grid-cols-2 sm:grid-cols-3' : 'grid-cols-2']">
          <div>
            <h4 class="footer-section-title mb-5 lg:mb-6">{{ t('footer.quickLinks') }}</h4>
            <ul class="space-y-2">
              <li v-for="item in quickLinks" :key="item.path">
                <router-link :to="item.path" class="footer-link-item group flex items-center gap-3 rounded-xl px-2.5 py-2.5 -mx-2 transition-all duration-200">
                  <div class="footer-link-icon flex items-center justify-center theme-text-muted transition-colors group-hover:text-blue-500 dark:group-hover:text-blue-300">
                    <svg class="footer-link-svg" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" :d="item.icon" />
                    </svg>
                  </div>
                  <span class="footer-link-label theme-link-muted group-hover:theme-text-primary">{{ t(item.label) }}</span>
                </router-link>
              </li>
            </ul>
          </div>

          <div>
            <h4 class="footer-section-title mb-5 lg:mb-6">{{ t('footer.legal') }}</h4>
            <ul class="space-y-2">
              <li>
                <router-link to="/privacy" class="footer-link-item group flex items-center gap-3 rounded-xl px-2.5 py-2.5 -mx-2 transition-all duration-200">
                  <div class="footer-link-icon flex items-center justify-center theme-text-muted transition-colors group-hover:text-emerald-500 dark:group-hover:text-emerald-300">
                    <svg class="footer-link-svg" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
                    </svg>
                  </div>
                  <span class="footer-link-label theme-link-muted group-hover:theme-text-primary">{{ t('footer.privacy') }}</span>
                </router-link>
              </li>
              <li>
                <router-link to="/terms" class="footer-link-item group flex items-center gap-3 rounded-xl px-2.5 py-2.5 -mx-2 transition-all duration-200">
                  <div class="footer-link-icon flex items-center justify-center theme-text-muted transition-colors group-hover:text-emerald-500 dark:group-hover:text-emerald-300">
                    <svg class="footer-link-svg" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                    </svg>
                  </div>
                  <span class="footer-link-label theme-link-muted group-hover:theme-text-primary">{{ t('footer.terms') }}</span>
                </router-link>
              </li>
              <li v-if="showAboutLink">
                <router-link to="/about" class="footer-link-item group flex items-center gap-3 rounded-xl px-2.5 py-2.5 -mx-2 transition-all duration-200">
                  <div class="footer-link-icon flex items-center justify-center theme-text-muted transition-colors group-hover:text-emerald-500 dark:group-hover:text-emerald-300">
                    <svg class="footer-link-svg" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                    </svg>
                  </div>
                  <span class="footer-link-label theme-link-muted group-hover:theme-text-primary">{{ t('nav.about') }}</span>
                </router-link>
              </li>
            </ul>
          </div>

          <div
            v-if="footerLinks.length"
            class="col-span-2 border-t border-dashed border-gray-200/60 pt-4 dark:border-white/10 sm:col-span-1 sm:border-none sm:pt-0"
          >
            <h4 class="footer-section-title mb-5 lg:mb-6">{{ t('footer.toolbox') }}</h4>
            <ul class="grid grid-cols-2 gap-2 sm:grid-cols-1">
              <li v-for="(link, index) in footerLinks" :key="link.url || `${link.name}-${index}`">
                <component
                  :is="link.url ? 'a' : 'div'"
                  :href="link.url ? link.url : undefined"
                  :target="link.url ? '_blank' : undefined"
                  :rel="link.url ? 'noopener noreferrer' : undefined"
                  class="footer-link-item group flex items-center gap-3 rounded-xl px-2.5 py-2.5 -mx-2 transition-all duration-200"
                  :class="{ 'cursor-default': !link.url }"
                >
                  <div
                    class="footer-link-icon footer-link-icon-letter flex shrink-0 items-center justify-center text-[11px] font-semibold leading-none theme-text-muted transition-colors group-hover:text-gray-900 dark:group-hover:text-white"
                  >
                    {{ getToolboxInitial(link.name) }}
                  </div>
                  <span class="footer-link-label theme-link-muted group-hover:theme-text-primary">{{ link.name }}</span>
                </component>
              </li>
            </ul>
          </div>
        </div>

        <div class="w-full space-y-4 lg:col-span-1 lg:ml-auto lg:max-w-[260px]">
          <h4 class="footer-section-title mb-4 lg:mb-6">{{ t('footer.contact') }}</h4>

          <div class="grid grid-cols-2 gap-3 lg:grid-cols-1">
            <a
              v-if="config?.contact?.telegram"
              :href="config.contact.telegram"
              target="_blank"
              rel="noopener noreferrer"
              class="footer-contact-card footer-contact-card-telegram group flex items-center justify-between rounded-2xl p-3.5"
            >
              <div class="flex items-center space-x-3">
                <div class="footer-contact-platform-icon footer-contact-platform-icon-telegram flex items-center justify-center text-white transition-transform duration-200 group-hover:scale-105">
                  <svg class="footer-contact-platform-glyph" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.14.18-.357.295-.6.295-.002 0-.003 0-.005 0l.213-3.054 5.56-5.022c.24-.213-.054-.334-.373-.121l-6.869 4.326-2.96-.924c-.64-.203-.658-.64.135-.954l11.566-4.458c.538-.196 1.006.128.832.941z" />
                  </svg>
                </div>
                <span class="footer-contact-label theme-text-primary">Telegram</span>
              </div>
              <svg class="footer-contact-arrow footer-contact-arrow-telegram h-4 w-4 transition-transform duration-200 group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </a>

            <a
              v-if="config?.contact?.whatsapp"
              :href="config.contact.whatsapp"
              target="_blank"
              rel="noopener noreferrer"
              class="footer-contact-card footer-contact-card-whatsapp group flex items-center justify-between rounded-2xl p-3.5"
            >
              <div class="flex items-center space-x-3">
                <div class="footer-contact-platform-icon footer-contact-platform-icon-whatsapp flex items-center justify-center text-white transition-transform duration-200 group-hover:scale-105">
                  <svg class="footer-contact-platform-glyph footer-contact-platform-glyph-whatsapp" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413Z" />
                  </svg>
                </div>
                <span class="footer-contact-label theme-text-primary">WhatsApp</span>
              </div>
              <svg class="footer-contact-arrow footer-contact-arrow-whatsapp h-4 w-4 transition-transform duration-200 group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </a>
          </div>
        </div>
      </div>

      <div class="mt-10 block border-t border-dashed border-gray-200/60 pt-6 text-center dark:border-white/10 lg:hidden">
        <p class="text-xs font-medium opacity-50">&copy; {{ currentYear }} {{ brandSiteName }}. {{ t('footer.rights') }}</p>
      </div>
    </div>
  </footer>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAppStore } from '../stores/app'

interface FooterLinkItem {
  name: string
  url?: string
}

const { t } = useI18n()
const appStore = useAppStore()

const config = computed(() => appStore.config)

const brandSiteName = computed(() => {
  const siteName = config.value?.brand?.site_name
  return typeof siteName === 'string' && siteName.trim() ? siteName.trim() : 'Dujiao-Next'
})

const brandDescription = computed(() => {
  const desc = config.value?.brand?.site_description
  if (desc && typeof desc === 'object') {
    const val = desc[appStore.locale] || desc['zh-CN'] || ''
    return typeof val === 'string' ? val.trim() : ''
  }
  return ''
})

const isListMode = computed(() => config.value?.template_mode === 'list')
const navConfig = computed(() => config.value?.nav_config as { builtin?: Record<string, boolean> } | undefined)

const quickLinks = computed(() => {
  const items = [
    { path: '/', label: 'nav.home', icon: 'M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-4 0a1 1 0 01-1-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 01-1 1' },
  ]
  if (!isListMode.value) {
    items.push({ path: '/products', label: 'nav.products', icon: 'M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z' })
  }
  const builtin = navConfig.value?.builtin
  if (!builtin || builtin.blog !== false) {
    items.push({ path: '/blog', label: 'nav.blog', icon: 'M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9a2 2 0 00-2-2h-2' })
  }
  return items
})

const showAboutLink = computed(() => {
  const builtin = navConfig.value?.builtin
  return !builtin || builtin.about !== false
})

const footerLinks = computed<FooterLinkItem[]>(() => {
  const links = config.value?.footer_links
  if (!Array.isArray(links)) return []
  return links.filter((item): item is FooterLinkItem => Boolean(item && typeof item.name === 'string' && item.name.trim()))
})

const getToolboxInitial = (name: string) => {
  const trimmed = typeof name === 'string' ? name.trim() : ''
  if (!trimmed) return '#'
  const first = Array.from(trimmed)[0] || '#'
  return /^[a-z]$/i.test(first) ? first.toUpperCase() : first
}

const currentYear = new Date().getFullYear()
</script>

<style scoped>
.footer-brand {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  min-height: 2.5rem;
  max-width: min(100%, 18rem);
  transition: transform 0.18s ease, opacity 0.18s ease;
}

.footer-brand:hover {
  transform: translateY(-0.5px);
  opacity: 0.98;
}

.footer-brand:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ui-focus-ring);
  border-radius: 1rem;
}

.footer-brand-logo {
  display: block;
  width: 1.92rem;
  height: 1.92rem;
  flex-shrink: 0;
  object-fit: contain;
  transition: transform 0.18s ease, opacity 0.18s ease;
}

.footer-brand:hover .footer-brand-logo {
  transform: translateY(-0.5px) scale(1.01);
  opacity: 0.96;
}

.footer-brand-name {
  --footer-brand-name-gradient: linear-gradient(
    180deg,
    color-mix(in oklab, white 70%, var(--ui-text-primary) 30%) 0%,
    color-mix(in oklab, white 36%, var(--ui-text-primary) 64%) 22%,
    color-mix(in oklab, var(--ui-text-primary) 86%, white 14%) 54%,
    color-mix(in oklab, var(--ui-text-primary) 94%, black 6%) 100%
  );
  --footer-brand-name-shadow: 0 1px 0 rgba(255, 255, 255, 0.12);
  --footer-brand-name-stroke: 0.2px color-mix(in oklab, var(--ui-text-primary) 72%, black 28%);
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: var(--ui-text-primary);
  font-size: clamp(1.02rem, 0.92rem + 0.28vw, 1.18rem);
  font-weight: 800;
  line-height: 1.02;
  letter-spacing: -0.03em;
  background-image: var(--footer-brand-name-gradient);
  text-shadow: var(--footer-brand-name-shadow);
  -webkit-text-stroke: var(--footer-brand-name-stroke);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}

:global(.dark) .footer-brand-name {
  --footer-brand-name-gradient: linear-gradient(
    180deg,
    color-mix(in oklab, white 94%, var(--ui-text-primary) 6%) 0%,
    color-mix(in oklab, white 76%, var(--ui-text-primary) 24%) 20%,
    color-mix(in oklab, white 52%, var(--ui-text-primary) 48%) 52%,
    color-mix(in oklab, var(--ui-text-primary) 82%, white 18%) 100%
  );
  --footer-brand-name-shadow: 0 1px 0 rgba(255, 255, 255, 0.08), 0 8px 18px rgba(0, 0, 0, 0.18);
  --footer-brand-name-stroke: 0.2px color-mix(in oklab, white 34%, var(--ui-text-primary) 66%);
}

.footer-section-title {
  display: block;
  color: var(--ui-text-primary);
  font-size: 0.74rem;
  font-weight: 700;
  line-height: 1;
  letter-spacing: 0.16em;
  text-transform: uppercase;
  opacity: 0.72;
}

.footer-link-item {
  position: relative;
  border: 1px solid transparent;
  background: transparent;
  transition:
    transform 0.2s ease,
    background-color 0.2s ease,
    border-color 0.2s ease,
    box-shadow 0.2s ease,
    color 0.2s ease;
}

.footer-link-item:hover {
  transform: translateY(-1px);
  background: color-mix(in oklab, var(--ui-bg-soft) 82%, transparent);
  border-color: color-mix(in oklab, var(--ui-border) 74%, transparent);
  box-shadow: 0 14px 26px -28px rgba(15, 23, 42, 0.22);
}

:global(.dark) .footer-link-item:hover {
  background: color-mix(in oklab, var(--ui-bg-overlay-strong) 88%, transparent);
  border-color: color-mix(in oklab, var(--ui-border) 84%, white 16%);
  box-shadow: 0 18px 30px -30px rgba(0, 0, 0, 0.5);
}

.footer-link-label {
  font-size: 0.82rem;
  font-weight: 500;
  letter-spacing: -0.01em;
  text-decoration: none !important;
}

.footer-link-label:hover,
.footer-link-label:focus-visible,
.footer-link-item:hover .footer-link-label,
.footer-link-item:focus-visible .footer-link-label {
  text-decoration: none !important;
}

.footer-link-icon {
  width: 1.62rem;
  height: 1.62rem;
  border-radius: 0.66rem;
  backdrop-filter: blur(14px);
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.88), rgba(248, 250, 252, 0.98)),
    rgba(255, 255, 255, 0.9);
  border: 1px solid color-mix(in oklab, var(--ui-border) 76%, white 24%);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.82),
    0 10px 22px -20px rgba(15, 23, 42, 0.24);
  transition:
    color 0.2s ease,
    background-color 0.2s ease,
    border-color 0.2s ease,
    box-shadow 0.2s ease,
    transform 0.2s ease;
}

.footer-link-svg {
  width: 0.76rem;
  height: 0.76rem;
  opacity: 0.84;
}

.footer-link-item:hover .footer-link-icon {
  transform: translateY(-1px) scale(1.02);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.88),
    0 14px 24px -20px rgba(15, 23, 42, 0.2);
}

.footer-link-item:hover .footer-link-svg {
  opacity: 0.96;
}

.footer-link-icon-letter {
  font-size: 0.56rem;
  font-weight: 700;
  letter-spacing: -0.01em;
}

:global(.dark) .footer-link-icon {
  backdrop-filter: none;
  background: var(--ui-bg-muted);
  border-color: color-mix(in oklab, var(--ui-border) 84%, transparent);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.02),
    0 8px 18px -14px rgba(0, 0, 0, 0.42);
}

:global(.dark) .group:hover .footer-link-icon {
  background: var(--ui-bg-soft);
  border-color: color-mix(in oklab, var(--ui-border-strong) 78%, transparent);
}

.footer-contact-card {
  position: relative;
  overflow: hidden;
  border: 1px solid color-mix(in oklab, var(--ui-border) 82%, white 18%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 90%, white 10%),
      color-mix(in oklab, var(--ui-bg-soft) 94%, transparent)
    );
  box-shadow: var(--ui-shadow-1);
  transition:
    transform 0.2s ease,
    border-color 0.2s ease,
    box-shadow 0.2s ease,
    background-color 0.2s ease;
}

.footer-contact-card:hover {
  transform: translateY(-1px);
}

:global(.dark) .footer-contact-card {
  border-color: color-mix(in oklab, var(--ui-border) 88%, white 12%);
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--ui-bg-elevated) 97%, black 3%),
      color-mix(in oklab, var(--ui-bg-muted) 94%, black 6%)
    );
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.03),
    0 18px 30px -30px rgba(0, 0, 0, 0.46);
}

.footer-contact-card-telegram:hover {
  border-color: color-mix(in oklab, rgb(59 130 246) 34%, var(--ui-border));
  box-shadow: 0 18px 32px -28px rgba(59, 130, 246, 0.24);
}

.footer-contact-card-whatsapp:hover {
  border-color: color-mix(in oklab, rgb(16 185 129) 34%, var(--ui-border));
  box-shadow: 0 18px 32px -28px rgba(16, 185, 129, 0.24);
}

.footer-contact-platform-icon {
  position: relative;
  width: 1.88rem;
  height: 1.88rem;
  border-radius: 0.72rem;
  border: 1px solid rgba(255, 255, 255, 0.14);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.18),
    0 10px 18px -16px rgba(15, 23, 42, 0.36);
}

.footer-contact-platform-icon-telegram {
  background: linear-gradient(180deg, rgba(59, 130, 246, 0.9), rgba(37, 99, 235, 0.76));
}

.footer-contact-platform-icon-whatsapp {
  background: linear-gradient(180deg, rgba(16, 185, 129, 0.88), rgba(5, 150, 105, 0.74));
}

.footer-contact-platform-glyph {
  width: 0.84rem;
  height: 0.84rem;
}

.footer-contact-platform-glyph-whatsapp {
  width: 0.9rem;
  height: 0.9rem;
}

.footer-contact-label {
  font-size: 0.82rem;
  font-weight: 600;
  letter-spacing: -0.01em;
}

.footer-contact-arrow {
  opacity: 0.78;
}

.footer-contact-arrow-telegram {
  color: rgba(96, 165, 250, 0.96);
}

.footer-contact-arrow-whatsapp {
  color: rgba(52, 211, 153, 0.96);
}
</style>
