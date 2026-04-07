<template>
  <section id="special_deals">
    <div class="list">
      <article
        v-for="(product, idx) in displayProducts"
        :key="`promo-${product.id}`"
        class="specialdeals theme-slide-up"
        :class="isSoldOut(product) ? 'is-disabled' : 'active'"
        :data-hue="getItemHueHex(product, idx)"
        :style="[getItemThemeVars(product, idx), { animationDelay: `${idx * 45}ms` }]"
        :tabindex="isSoldOut(product) ? -1 : 0"
        @click="openProduct(product)"
        @keydown.enter.prevent="openProduct(product)"
        @keydown.space.prevent="openProduct(product)"
      >
        <div class="item">
          <div class="img">
            <img
              v-if="getProductImage(product)"
              class="IconPic"
              width="54"
              height="54"
              :src="getProductImage(product)"
              :alt="getLocalizedText(product.title)"
              crossorigin="anonymous"
              loading="lazy"
              :ref="(el) => setImageRef(product, idx, el as Element | null)"
              @load="handleImageLoad(product, idx, $event)"
            >
            <div v-else class="fallback-icon">
              <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
            </div>
          </div>
          <div class="T">
            <div class="sku">{{ getProductNameLine(product) }}</div>
            <div class="name">{{ getProductDescLine(product) }}</div>
          </div>
        </div>

        <div class="promo">
          <div class="rate">{{ promotionLabel }}</div>
          <div class="price">
            <span class="price-copy">{{ getPromotionCopy(product) }}</span>
          </div>
        </div>

        <div v-if="isSoldOut(product)" class="soldout-mask">
          <span class="soldout-tag">{{ t('products.stockStatus.outOfStock') }}</span>
        </div>
      </article>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed, reactive, watch, type CSSProperties } from 'vue'
import { useI18n } from 'vue-i18n'
import { getFirstImageUrl, getImageUrl } from '../utils/image'
import { amountToCents } from '../utils/money'
import { useLocalized, useProductLabels } from '../composables/useProduct'

const props = defineProps<{
  products: any[]
}>()

const emit = defineEmits<{
  click: [slug: string]
}>()

const { t, locale } = useI18n()
const { getLocalizedText, siteCurrency, formatPrice } = useLocalized()
const { isSoldOut, hasPromotionPrice, getPromotionPriceAmount, getPromotionRules } = useProductLabels()

const isZhLocale = computed(() => String(locale.value || '').toLowerCase().startsWith('zh'))
const promotionLabel = computed(() => (isZhLocale.value ? '促销' : 'Promo'))
const displayProducts = computed(() => (Array.isArray(props.products) ? props.products.slice(0, 8) : []))
const imageHueMap = reactive<Record<string, string>>({})

const defaultHuePalette = [
  '757a98', '3370a1', '141b24', '585556', '517198',
  '0e4cae', 'c545f8', 'cd6b9f', '686d6d', '678c97',
]

const hashText = (text: string) => {
  let hash = 0
  for (let i = 0; i < text.length; i += 1) {
    hash = (hash << 5) - hash + text.charCodeAt(i)
    hash |= 0
  }
  return Math.abs(hash)
}

const normalizeHex = (input: unknown) => {
  const raw = String(input || '').trim().replace(/^#/, '')
  if (!raw) return ''
  if (/^[0-9a-fA-F]{6}$/.test(raw)) return raw.toLowerCase()
  if (/^[0-9a-fA-F]{3}$/.test(raw)) {
    return raw.toLowerCase().split('').map((s) => `${s}${s}`).join('')
  }
  return ''
}

const getProductKey = (product: any, index: number) => String(product?.id ?? product?.slug ?? `promo-${index}`)

const getFallbackHueHex = (product: any, index: number) => {
  const explicitHex = normalizeHex(product?.promotion_hue || product?.special_deal_hue || product?.theme_hue)
  if (explicitHex) return explicitHex
  const seedText = `${getLocalizedText(product?.title)}-${product?.id || index}`
  const paletteIndex = hashText(seedText) % defaultHuePalette.length
  return defaultHuePalette[paletteIndex] || defaultHuePalette[index % defaultHuePalette.length] || '757a98'
}

const getItemHueHex = (product: any, index: number) => {
  const key = getProductKey(product, index)
  return imageHueMap[key] || getFallbackHueHex(product, index)
}

const hexToRgb = (hex: string) => {
  const normalized = normalizeHex(hex)
  const value = Number.parseInt(normalized || '000000', 16)
  return {
    r: (value >> 16) & 255,
    g: (value >> 8) & 255,
    b: value & 255,
  }
}

const rgbToHsl = (r: number, g: number, b: number) => {
  const nr = r / 255
  const ng = g / 255
  const nb = b / 255
  const max = Math.max(nr, ng, nb)
  const min = Math.min(nr, ng, nb)
  const delta = max - min

  let h = 0
  if (delta !== 0) {
    if (max === nr) h = ((ng - nb) / delta) % 6
    else if (max === ng) h = (nb - nr) / delta + 2
    else h = (nr - ng) / delta + 4
  }

  h = Math.round(h * 60)
  if (h < 0) h += 360

  const l = (max + min) / 2
  const s = delta === 0 ? 0 : delta / (1 - Math.abs(2 * l - 1))
  return {
    h,
    s: Number((s * 100).toFixed(2)),
    l: Number((l * 100).toFixed(2)),
  }
}

const getItemThemeVars = (product: any, index: number): CSSProperties => {
  const hex = getItemHueHex(product, index)
  const { r, g, b } = hexToRgb(hex)
  const { h, s, l } = rgbToHsl(r, g, b)

  return {
    '--d-r': String(r),
    '--d-g': String(g),
    '--d-b': String(b),
    '--d-h': String(h),
    '--d-s': `${s}%`,
    '--d-l': `${l}%`,
    '--background-alpha': '1',
  } as CSSProperties
}

const getProductImage = (product: any) => {
  if (product?.images) {
    const image = getFirstImageUrl(product.images)
    if (image) return image
  }
  return getImageUrl(product?.category?.icon)
}

const getProductNameLine = (product: any) => {
  const title = getLocalizedText(product?.title)
  if (title) return title
  return t('products.promotionBadge')
}

const getProductDescLine = (product: any) => {
  const description = String(getLocalizedText(product?.description) || '').trim()
  return description
}

const rgbToHex = (r: number, g: number, b: number) =>
  [r, g, b]
    .map((value) => Math.max(0, Math.min(255, Math.round(value))).toString(16).padStart(2, '0'))
    .join('')

const extractDominantImageHue = (img: HTMLImageElement) => {
  if (typeof document === 'undefined') return ''

  const canvas = document.createElement('canvas')
  canvas.width = 20
  canvas.height = 20

  const context = canvas.getContext('2d', { willReadFrequently: true })
  if (!context) return ''

  try {
    context.drawImage(img, 0, 0, canvas.width, canvas.height)
  } catch {
    return ''
  }

  let data: Uint8ClampedArray
  try {
    data = context.getImageData(0, 0, canvas.width, canvas.height).data
  } catch {
    return ''
  }

  let vividWeight = 0
  let vividR = 0
  let vividG = 0
  let vividB = 0

  let fallbackWeight = 0
  let fallbackR = 0
  let fallbackG = 0
  let fallbackB = 0

  for (let i = 0; i < data.length; i += 4) {
    const alpha = (data[i + 3] ?? 0) / 255
    if (alpha < 0.15) continue

    const r = data[i] ?? 0
    const g = data[i + 1] ?? 0
    const b = data[i + 2] ?? 0
    const { s, l } = rgbToHsl(r, g, b)

    fallbackWeight += alpha
    fallbackR += r * alpha
    fallbackG += g * alpha
    fallbackB += b * alpha

    const ignorePixel = (s < 10 && l > 92) || l < 6
    if (ignorePixel) continue

    const weight = alpha * (0.6 + s / 100)
    vividWeight += weight
    vividR += r * weight
    vividG += g * weight
    vividB += b * weight
  }

  if (vividWeight > 0) {
    return rgbToHex(vividR / vividWeight, vividG / vividWeight, vividB / vividWeight)
  }

  if (fallbackWeight > 0) {
    return rgbToHex(fallbackR / fallbackWeight, fallbackG / fallbackWeight, fallbackB / fallbackWeight)
  }

  return ''
}

const applyImageHue = (product: any, index: number, img: HTMLImageElement | null) => {
  if (!img) return
  const key = getProductKey(product, index)
  const extractedHue = extractDominantImageHue(img)
  imageHueMap[key] = extractedHue || getFallbackHueHex(product, index)
}

const setImageRef = (product: any, index: number, el: Element | null) => {
  if (!(el instanceof HTMLImageElement)) return
  if (el.complete && el.naturalWidth > 0) {
    applyImageHue(product, index, el)
  }
}

const handleImageLoad = (product: any, index: number, event: Event) => {
  const target = event.target
  if (!(target instanceof HTMLImageElement)) return
  applyImageHue(product, index, target)
}

watch(
  displayProducts,
  (items) => {
    const activeKeys = new Set(items.map((product, index) => getProductKey(product, index)))
    Object.keys(imageHueMap).forEach((key) => {
      if (!activeKeys.has(key)) delete imageHueMap[key]
    })
  },
  { immediate: true },
)

const capPercent = (value: number) => Math.min(99.9, Math.max(0, value))

const getDiscountPercent = (product: any): number | null => {
  const original = amountToCents(product?.price_amount)
  if (original === null || original <= 0) return null

  if (hasPromotionPrice(product)) {
    const promo = amountToCents(getPromotionPriceAmount(product))
    if (promo !== null && promo >= 0 && promo < original) {
      return capPercent(((original - promo) / original) * 100)
    }
  }

  const rules = getPromotionRules(product)
  if (!Array.isArray(rules)) return null

  const percentRule = rules.find((rule) => rule?.type === 'percent' && Number(rule?.value) > 0)
  if (percentRule) return capPercent(Number(percentRule.value))

  const fixedRule = rules.find((rule) => rule?.type === 'fixed' && Number(rule?.value) > 0)
  if (fixedRule) {
    const saveCents = amountToCents(fixedRule.value)
    if (saveCents !== null && saveCents > 0) {
      return capPercent((saveCents / original) * 100)
    }
  }

  const specialRule = rules.find((rule) => rule?.type === 'special_price')
  if (specialRule) {
    const specialPrice = amountToCents(specialRule.value)
    if (specialPrice !== null && specialPrice >= 0 && specialPrice < original) {
      return capPercent(((original - specialPrice) / original) * 100)
    }
  }

  return null
}

const getDiscountLabel = (product: any) => {
  const discountPercent = getDiscountPercent(product)
  if (discountPercent === null) return '-0.0%'
  return `-${discountPercent.toFixed(1)}%`
}

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
      return String(rule?.name || '').trim()
  }
}

const getPromotionCopy = (product: any) => {
  const rules = getPromotionRules(product)
  if (Array.isArray(rules) && rules.length > 0) {
    const firstRuleText = formatPromotionRule(rules[0])
    if (firstRuleText) return firstRuleText
  }

  if (hasPromotionPrice(product)) {
    const value = formatPrice(getPromotionPriceAmount(product), siteCurrency.value)
    return t('products.promotionHintSpecialNoMin', { value })
  }

  return getDiscountLabel(product)
}

const openProduct = (product: any) => {
  if (isSoldOut(product) || !product?.slug) return
  emit('click', product.slug)
}
</script>

<style scoped>
@font-face {
  font-family: 'SeagmManrope';
  src: url('https://www.seagm.com/skin/fonts/manrope/v15/manrope-8951283ba1.woff2') format('woff2');
  font-weight: 100 900;
  font-style: normal;
  font-display: swap;
}

#special_deals {
  --SiteFS: 12px;
  --SiteFS-S: 0.833334em;
  --SiteFW-B: 600;
  --background-color-L3: rgba(255, 255, 255, 0.5);
  --body-background-color: #fff;
  --text-color-promotion: #279737;
  --angle-reverse: -1;
  position: relative;
  touch-action: pan-y;
  display: flex;
  flex-direction: column;
  gap: 0;
  font-family: 'SeagmManrope', 'Manrope', 'Arial', 'Verdana', 'PingFang', 'Helvetica', system-ui, sans-serif;
  font-size: var(--SiteFS);
  line-height: 1.5;
}

.list {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  grid-auto-rows: 1fr;
  gap: 2em 1em;
}

.specialdeals {
  display: flex;
  flex-direction: column;
  position: relative;
  height: 100%;
  color: inherit;
  overflow: visible;
  border-radius: 1em;
  background-color: var(--background-color-L3, rgba(255 255 255 / 50%));
  border: 1px solid rgba(148, 163, 184, 0.12);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.24),
    0 0.95rem 1.8rem -1.5rem rgba(15, 23, 42, 0.2);
  transition: opacity 300ms, transform 240ms ease;
}

.specialdeals.active {
  cursor: pointer;
}

.specialdeals.is-disabled {
  cursor: not-allowed;
  opacity: 0.65;
}

.item {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 1em;
  align-items: center;
  padding: 1em;
  border-radius: 1em 1em 0 0;
  color: var(--color);
  background-color: var(--background-color);
  background-image:
    url('https://www.seagm.com/skin/images/art/gradient_bg-f5a8237834.svg'),
    linear-gradient(
      calc(var(--angle-reverse, 1) * 135deg),
      hsl(var(--offset-h1) var(--offset-s1) var(--offset-l1)),
      hsl(var(--offset-h2) var(--offset-s2) var(--offset-l2))
    );
  background-repeat: repeat-x;
  background-position: -90% 0, center;
  background-size: 225% auto, cover;
}

.item .img {
  width: 5em;
  height: 5em;
  border-radius: 25%;
  overflow: hidden;
  background-color: var(--body-background-color, #fff);
  box-shadow: var(--color-img-bdc) 0 0 0 1px;
  margin-block-start: -2em;
}

.item .img img {
  display: block;
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100%;
  object-fit: cover;
}

.item .T {
  display: flex;
  flex-direction: column;
  gap: 0.25em;
  overflow: hidden;
}

.item .sku {
  font-weight: var(--SiteFW-B);
}

.item .name {
  font-size: var(--SiteFS-S);
  opacity: 0.75;
  min-height: 1.25em;
}

.item .sku,
.item .name {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.promo {
  display: grid;
  grid-template-columns: max-content minmax(0, 1fr);
  gap: 0.4em;
  align-items: center;
  padding: 0.72em 0.82em;
  border-radius: 0 0 1em 1em;
  background:
    linear-gradient(
      180deg,
      rgba(var(--r) var(--g) var(--b) / 0.18),
      rgba(var(--r) var(--g) var(--b) / 0.08)
    );
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.18);
}

.price {
  margin-inline-start: auto;
  min-width: 0;
  width: 100%;
  max-width: none;
  text-align: right;
  font-size: 0.84em;
  line-height: 1.2;
}

.price-copy {
  display: block;
  font-weight: var(--SiteFW-B);
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.rate {
  background:
    linear-gradient(
      180deg,
      color-mix(in oklab, var(--text-color-promotion) 14%, white 86%),
      color-mix(in oklab, var(--text-color-promotion) 18%, white 82%)
    );
  color: color-mix(in oklab, var(--text-color-promotion) 74%, #0f172a 26%);
  min-width: 3.35em;
  padding-inline: 0.28em;
  box-sizing: border-box;
  border: 1px solid color-mix(in oklab, var(--text-color-promotion) 18%, white 82%);
  font-weight: var(--SiteFW-B);
  text-align: center;
  border-radius: 0.5em;
  line-height: 1.6;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.42);
}

.soldout-mask {
  position: absolute;
  inset: 0;
  z-index: 5;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(15, 23, 42, 0.4);
}

.soldout-tag {
  border-radius: 0.5rem;
  background: #ef4444;
  color: #fff;
  font-size: 0.72rem;
  font-weight: 700;
  padding: 0.35rem 0.65rem;
}

.fallback-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  color: rgba(0, 0, 0, 0.45);
}

.specialdeals[data-hue] {
  --r: var(--d-r, 255);
  --g: var(--d-g, 255);
  --b: var(--d-b, 255);
  --h: var(--d-h, 0);
  --s: var(--d-s, 0%);
  --l: var(--d-l, 100%);
  --threshold: 0.6;
  --background-color: rgba(var(--r) var(--g) var(--b) / var(--background-alpha, 1));
  --red: calc(var(--r) * 0.2126);
  --green: calc(var(--g) * 0.7152);
  --blue: calc(var(--b) * 0.0722);
  --sum: calc(var(--red) + var(--green) + var(--blue));
  --lightness: calc(var(--sum) / 255);
  --lightness-trend: clamp(0%, calc((var(--lightness) - var(--threshold)) * -999999%), 100%);
  --color: hsl(0 0% var(--lightness-trend));
  --color-img-bdc: hsl(0 0% var(--lightness-trend) / 10%);
  --offset-h1: calc(var(--h) + 10);
  --offset-h2: calc(var(--h) - 10);
  --offset-s1: calc(var(--s) * 0.78);
  --offset-s2: calc(var(--s) * 0.82);
  --offset-l1: max(calc(var(--l) - 8%), 22%);
  --offset-l2: min(calc(var(--l) + 7%), 76%);
}

@media only screen and (max-width: 1000px), (hover: none) and (pointer: coarse) {
  #special_deals {
    font-size: 13px;
  }

  #special_deals > .list {
    display: flex !important;
    flex-direction: column;
    flex-wrap: wrap;
    gap: 1em 0;
    max-height: 20em;
    overflow-x: auto;
    overflow-y: hidden;
    margin: 0;
    padding: 0.75rem;
    scroll-padding-inline: 0.75rem;
    -webkit-overflow-scrolling: touch;
  }

  #special_deals > .list::-webkit-scrollbar {
    display: none;
  }

  #special_deals > .list > .specialdeals {
    flex: none;
    width: min(22.5em, calc(100% - 1.5rem));
  }

  .item {
    gap: 0.62em;
    padding: 0.72em;
  }

  .item .img {
    width: 3.95em;
    height: 3.95em;
    border-radius: 28%;
    margin-block-start: -1.45em;
    box-shadow:
      0 0 0 1px var(--color-img-bdc),
      0 0.55rem 1.1rem -0.6rem rgba(15, 23, 42, 0.55);
    transform: translateY(-1px);
  }

  .item .img img {
    transform: scale(1.04);
  }

  .item .sku {
    font-size: 1.02em;
    line-height: 1.25;
  }

  .item .name {
    min-height: 1.35em;
  }

  .promo {
    gap: 0.28em;
    padding: 0.64em 0.58em;
  }

  .rate {
    min-width: 2.88em;
    padding-inline: 0.18em;
    font-size: 0.88em;
  }

  .price {
    width: 100%;
    max-width: none;
    font-size: 0.78em;
  }
}

:global(.dark #special_deals) {
  --background-color-L3: #0c1016;
  --body-background-color: #11161d;
  --text-color-promotion: #22c55e;
}

:global(.dark .specialdeals) {
  border: 1px solid rgba(148, 163, 184, 0.12);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.04),
    0 0.85rem 1.7rem -1.35rem rgba(0, 0, 0, 0.56);
}

:global(.dark .specialdeals[data-hue]) {
  --offset-s1: calc(var(--s) * 0.46);
  --offset-s2: calc(var(--s) * 0.5);
  --offset-l1: max(calc(var(--l) - 19%), 18%);
  --offset-l2: min(calc(var(--l) + 1%), 38%);
}

:global(.dark .item) {
  background-image:
    url('https://www.seagm.com/skin/images/art/gradient_bg-f5a8237834.svg'),
    linear-gradient(
      calc(var(--angle-reverse, 1) * 135deg),
      hsl(var(--offset-h1) var(--offset-s1) var(--offset-l1)),
      hsl(var(--offset-h2) var(--offset-s2) var(--offset-l2))
    );
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

:global(.dark .item .name) {
  opacity: 0.82;
}

:global(.dark .promo) {
  color: rgba(241, 245, 249, 0.95);
  background:
    linear-gradient(
      180deg,
      rgba(var(--r) var(--g) var(--b) / 0.08),
      rgba(10, 13, 18, 0.94) 82%
    );
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.05),
    inset 0 -1px 0 rgba(255, 255, 255, 0.02);
}

:global(.dark .rate) {
  background:
    linear-gradient(
      180deg,
      rgba(34, 197, 94, 0.2),
      rgba(34, 197, 94, 0.14)
    );
  color: rgba(220, 252, 231, 0.95);
  border-color: rgba(34, 197, 94, 0.2);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.06);
}

:global(.dark .price) {
  color: rgba(248, 250, 252, 0.96);
}
</style>
