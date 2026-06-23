<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

/**
 * Welcome screen for Vivian — Northwest University Marketing.
 *
 * Self-contained, single-page experience:
 *   - Cinematic 3-image slideshow (mountain / flowers / sunset)
 *   - Ken Burns motion on each slide
 *   - Slow crossfades (1.4s)
 *   - Glassmorphism welcome panel
 *   - Quote reveal with staggered timing
 *   - Brand chip + slide counter
 *   - Gold rule accents + soft glow
 *
 * Uses `definePageMeta({ layout: 'blank' })` to skip the default
 * shell (hamburger menu, world nav) — this is a standalone
 * welcome screen, not a world.
 *
 * Headline: "Welcome, Vivian"
 * Subheadline: "Photographer • Videographer • Storyteller"
 * Quote: "Photography captures a moment. Videography brings it to life."
 * Footer: "We're excited to welcome Vivian to the Northwest University Marketing Team."
 */

type Slide = { src: string; alt: string; ken: 'tl' | 'br' | 'tr' }

definePageMeta({ layout: 'blank' })

const slides: Slide[] = [
  { src: '/images/vivian-mountain.jpg', alt: 'Vivian — mountain landscape', ken: 'tl' },
  { src: '/images/vivian-flowers.jpg',  alt: 'Vivian — flowers in soft light', ken: 'br' },
  { src: '/images/vivian-sunset.jpg',   alt: 'Vivian — sunset horizon',       ken: 'tr' },
]

const active = ref(0)
const next = computed(() => (active.value + 1) % slides.length)
const titleVisible = ref(false)
const subVisible = ref(false)
const quoteVisible = ref(false)
const footerVisible = ref(false)

const CYCLE_MS = 7000
let cycleId: ReturnType<typeof setInterval> | null = null
let revealId: ReturnType<typeof setTimeout> | null = null

function nextSlide() {
  active.value = (active.value + 1) % slides.length
  // Replay text reveals on every transition for a cinema feel
  titleVisible.value = false
  subVisible.value = false
  quoteVisible.value = false
  footerVisible.value = false
  if (revealId) clearTimeout(revealId)
  revealId = setTimeout(() => {
    titleVisible.value = true
    setTimeout(() => { subVisible.value = true }, 700)
    setTimeout(() => { quoteVisible.value = true }, 1500)
    setTimeout(() => { footerVisible.value = true }, 2200)
  }, 1200)
}

onMounted(() => {
  // Initial reveals (first slide is already visible)
  setTimeout(() => { titleVisible.value = true }, 600)
  setTimeout(() => { subVisible.value = true }, 1200)
  setTimeout(() => { quoteVisible.value = true }, 1900)
  setTimeout(() => { footerVisible.value = true }, 2600)

  cycleId = setInterval(nextSlide, CYCLE_MS)
})

onUnmounted(() => {
  if (cycleId) clearInterval(cycleId)
  if (revealId) clearTimeout(revealId)
})
</script>

<template>
  <div class="vivian-screen" role="region" aria-label="Welcome Vivian">
    <!-- Cinematic image stack: all three always present, crossfaded -->
    <div class="stage">
      <div
        v-for="(s, i) in slides"
        :key="s.src"
        class="slide"
        :class="[
          { 'is-active': i === active, 'is-next': i === next },
          `ken-${s.ken}`,
        ]"
        :aria-hidden="i !== active"
      >
        <div class="slide-img" :style="{ backgroundImage: `url(${s.src})` }" />
        <!-- Soft navy → midnight gradient over the image so the
             foreground text always reads against the dark palette. -->
        <div class="slide-veil" />
        <div class="slide-glow" />
      </div>
    </div>

    <!-- Top-left: NU monogram chip (uses CSS only — no asset). -->
    <div class="brand-mark" :class="{ 'is-in': footerVisible }">
      <span class="brand-glyph">NU</span>
      <span class="brand-divider" />
      <span class="brand-word">Northwest University · Marketing</span>
    </div>

    <!-- Top-right: rotating slide counter, gives the slideshow
         a sense of progression without being loud. -->
    <div class="counter" :class="{ 'is-in': footerVisible }">
      <span class="counter-label">Visual</span>
      <span class="counter-current">{{ String(active + 1).padStart(2, '0') }}</span>
      <span class="counter-sep">/</span>
      <span class="counter-total">{{ String(slides.length).padStart(2, '0') }}</span>
    </div>

    <!-- Center: gold rule + headline + subheadline + quote -->
    <main class="hero" :class="{ 'is-in': titleVisible }">
      <div class="eyebrow">
        <span class="eyebrow-line" />
        <span class="eyebrow-text">Welcome to the team</span>
        <span class="eyebrow-line" />
      </div>
      <h1 class="title" :class="{ 'is-in': titleVisible }">
        <span class="title-word title-word--first">Welcome,</span>
        <span class="title-word title-word--second">Vivian</span>
      </h1>
      <p class="subtitle" :class="{ 'is-in': subVisible }">
        <span class="role">Photographer</span>
        <span class="bullet" aria-hidden="true">•</span>
        <span class="role">Videographer</span>
        <span class="bullet" aria-hidden="true">•</span>
        <span class="role">Storyteller</span>
      </p>

      <figure class="quote" :class="{ 'is-in': quoteVisible }">
        <span class="quote-mark" aria-hidden="true">&ldquo;</span>
        <blockquote class="quote-body">
          Photography captures a moment.<br>
          Videography brings it to life.
        </blockquote>
        <figcaption class="quote-cite">— Vivian's craft, in her own words</figcaption>
      </figure>
    </main>

    <!-- Bottom: glassmorphism welcome panel with gold accent line. -->
    <footer class="footer" :class="{ 'is-in': footerVisible }">
      <div class="footer-card">
        <span class="footer-accent" aria-hidden="true" />
        <div class="footer-text">
          <div class="footer-line">We're excited to welcome Vivian to the</div>
          <div class="footer-line footer-line--strong">Northwest University Marketing Team.</div>
        </div>
        <div class="footer-meta">
          <span class="meta-dot" />
          <span>Now onboarding</span>
        </div>
      </div>
    </footer>

    <!-- Decorative edge hairline that picks up the gold rule at the
         bottom — gives the composition a frame. -->
    <div class="edge-rule" aria-hidden="true" />
  </div>
</template>

<style scoped>
/* ============================================================
 * Color tokens — locked NU palette.
 * ============================================================ */
.vivian-screen {
  --nu-blue: #0068bb;
  --nu-navy: #034c87;
  --nu-midnight: #00263d;
  --nu-tour: #fbd945;        /* signature gold */
  --nu-amber: #ffbc2d;        /* warmer gold accent */
  --nu-leaf: #44ba82;        /* subtle support */
  --nu-wisp: #f4f8fc;        /* near-white */
  --nu-skylight: #bcd2e6;    /* cool off-white */

  position: fixed;
  inset: 0;
  background: var(--nu-midnight);
  color: var(--nu-wisp);
  overflow: hidden;
  font-family: var(--font-sans, 'Inter', system-ui, sans-serif);
  z-index: 1;
}

/* Subtle vignette on top of the image stack for extra depth */
.vivian-screen::after {
  content: '';
  position: absolute;
  inset: 0;
  background:
    radial-gradient(ellipse at 50% 110%, rgba(0, 0, 0, 0.55) 0%, transparent 60%),
    radial-gradient(ellipse at 50% -10%, rgba(0, 47, 75, 0.45) 0%, transparent 55%);
  pointer-events: none;
  z-index: 1;
}

/* ============================================================
 * Image stack — 3 slides, always present. Active is opaque,
 * the rest are transparent. Crossfades are slow (1.4s) so the
 * slideshow feels cinematic.
 * ============================================================ */
.stage {
  position: absolute;
  inset: 0;
  z-index: 0;
}

.slide {
  position: absolute;
  inset: 0;
  opacity: 0;
  transition: opacity 1.4s cubic-bezier(0.22, 1, 0.36, 1);
  will-change: opacity;
}

.slide.is-active {
  opacity: 1;
  z-index: 2;
}

.slide-img {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  transform-origin: center;
  /* Ken Burns — every slide zooms slowly inward (12s) and pans
     in a different direction. The next slide pre-positions itself
     so the crossfade doesn't pop. */
  animation: kenBurns 12s ease-in-out infinite alternate;
  will-change: transform;
}

.is-active.ken-tl .slide-img { animation-name: kenTL }
.is-active.ken-br .slide-img { animation-name: kenBR }
.is-active.ken-tr .slide-img { animation-name: kenTR }

/* Next slide sits behind, slightly shifted, so when active flips
   the crossfade has subtle direction. */
.is-next.ken-tl .slide-img { transform: scale(1.05) translate(-1.5%, -1.5%) }
.is-next.ken-br .slide-img { transform: scale(1.05) translate(1.5%, 1.5%) }
.is-next.ken-tr .slide-img { transform: scale(1.05) translate(1.5%, -1.5%) }

@keyframes kenTL {
  0%   { transform: scale(1.08) translate(-1.5%, -1.5%) }
  100% { transform: scale(1.18) translate(1.5%, 1.5%) }
}
@keyframes kenBR {
  0%   { transform: scale(1.08) translate(1.5%, 1.5%) }
  100% { transform: scale(1.18) translate(-1.5%, -1.5%) }
}
@keyframes kenTR {
  0%   { transform: scale(1.08) translate(1.5%, -1.5%) }
  100% { transform: scale(1.18) translate(-1.5%, 1.5%) }
}

/* Two layered veils: one dark (legibility) and one gold (warmth). */
.slide-veil {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(
      180deg,
      rgba(0, 38, 61, 0.55) 0%,
      rgba(0, 38, 61, 0.25) 28%,
      rgba(0, 38, 61, 0.55) 68%,
      rgba(0, 20, 36, 0.85) 100%
    );
}

.slide-glow {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(ellipse at 20% 22%, rgba(251, 217, 69, 0.12) 0%, transparent 50%),
    radial-gradient(ellipse at 80% 78%, rgba(0, 104, 187, 0.18) 0%, transparent 55%);
  mix-blend-mode: screen;
}

/* ============================================================
 * Brand mark (top-left) and counter (top-right)
 * ============================================================ */
.brand-mark {
  position: absolute;
  top: 56px;
  left: 64px;
  z-index: 4;
  display: inline-flex;
  align-items: center;
  gap: 18px;
  padding: 14px 22px 14px 18px;
  background: rgba(0, 38, 61, 0.42);
  border: 1px solid rgba(251, 217, 69, 0.22);
  border-radius: 999px;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  opacity: 0;
  transform: translateY(-12px);
  transition: opacity 0.8s ease, transform 0.8s ease;
}
.brand-mark.is-in {
  opacity: 1;
  transform: translateY(0);
}

.brand-glyph {
  font-family: var(--font-serif, 'Fraunces', Georgia, serif);
  font-size: 22px;
  font-weight: 800;
  letter-spacing: 0.04em;
  color: var(--nu-tour);
  background: linear-gradient(180deg, #fff5b8 0%, #fbd945 55%, #e6b41a 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  line-height: 1;
}

.brand-divider {
  width: 1px;
  height: 22px;
  background: rgba(251, 217, 69, 0.4);
}

.brand-word {
  font-size: 14px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  font-weight: 600;
  color: var(--nu-skylight);
}

.counter {
  position: absolute;
  top: 64px;
  right: 64px;
  z-index: 4;
  display: inline-flex;
  align-items: baseline;
  gap: 8px;
  padding: 10px 18px;
  background: rgba(0, 38, 61, 0.42);
  border: 1px solid rgba(251, 217, 69, 0.22);
  border-radius: 999px;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  opacity: 0;
  transform: translateY(-12px);
  transition: opacity 0.8s ease 0.1s, transform 0.8s ease 0.1s;
  font-feature-settings: 'tnum' 1;
}
.counter.is-in {
  opacity: 1;
  transform: translateY(0);
}
.counter-label {
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--nu-skylight);
  font-weight: 600;
}
.counter-current {
  font-family: var(--font-serif, 'Fraunces', Georgia, serif);
  font-size: 28px;
  font-weight: 700;
  color: var(--nu-tour);
  line-height: 1;
  margin-left: 6px;
}
.counter-sep {
  color: rgba(251, 217, 69, 0.5);
  font-size: 22px;
}
.counter-total {
  font-family: var(--font-serif, 'Fraunces', Georgia, serif);
  font-size: 20px;
  font-weight: 600;
  color: var(--nu-skylight);
  line-height: 1;
}

/* ============================================================
 * Hero (centered) — eyebrow + title + subtitle + quote
 * ============================================================ */
.hero {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  z-index: 3;
  transform: translateY(-58%);
  text-align: center;
  padding: 0 96px;
  opacity: 0;
  transition: opacity 0.9s ease 0.15s;
}
.hero.is-in {
  opacity: 1;
}

.eyebrow {
  display: inline-flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 36px;
}
.eyebrow-line {
  width: 56px;
  height: 1px;
  background: linear-gradient(90deg, transparent 0%, var(--nu-tour) 50%, transparent 100%);
}
.eyebrow-text {
  font-size: 16px;
  letter-spacing: 0.4em;
  text-transform: uppercase;
  color: var(--nu-tour);
  font-weight: 600;
}

.title {
  font-family: var(--font-serif, 'Fraunces', Georgia, serif);
  font-weight: 700;
  font-size: 200px;
  line-height: 0.96;
  margin: 0 0 36px 0;
  letter-spacing: -0.01em;
}
.title-word {
  display: inline-block;
  color: var(--nu-wisp);
  text-shadow: 0 6px 36px rgba(0, 0, 0, 0.45);
  opacity: 0;
  transform: translateY(28px);
  filter: blur(12px);
  transition: opacity 1.2s cubic-bezier(0.22, 1, 0.36, 1),
              transform 1.2s cubic-bezier(0.22, 1, 0.36, 1),
              filter 1.2s cubic-bezier(0.22, 1, 0.36, 1);
}
.title-word--first {
  margin-right: 28px;
  font-weight: 400;
  font-style: italic;
  color: var(--nu-skylight);
}
.title-word--second {
  font-weight: 800;
  background: linear-gradient(180deg, #fff8d0 0%, #fbd945 50%, #d99a0a 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 4px 24px rgba(251, 217, 69, 0.32));
  transition-delay: 0.15s;
}
.title.is-in .title-word {
  opacity: 1;
  transform: translateY(0);
  filter: blur(0);
}

.subtitle {
  font-size: 32px;
  font-weight: 500;
  letter-spacing: 0.04em;
  color: var(--nu-skylight);
  margin: 0 auto 64px;
  display: inline-flex;
  align-items: center;
  gap: 18px;
  flex-wrap: wrap;
  justify-content: center;
  max-width: 1100px;
  opacity: 0;
  transform: translateY(20px);
  filter: blur(6px);
  transition: opacity 1.1s ease, transform 1.1s ease, filter 1.1s ease;
}
.subtitle.is-in {
  opacity: 1;
  transform: translateY(0);
  filter: blur(0);
}
.role {
  white-space: nowrap;
}
.bullet {
  color: var(--nu-tour);
  font-size: 22px;
  line-height: 1;
  opacity: 0.85;
}

.quote {
  margin: 0 auto;
  max-width: 1180px;
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  opacity: 0;
  transform: translateY(18px);
  filter: blur(8px);
  transition: opacity 1.1s ease, transform 1.1s ease, filter 1.1s ease;
}
.quote.is-in {
  opacity: 1;
  transform: translateY(0);
  filter: blur(0);
}
.quote-mark {
  font-family: var(--font-serif, 'Fraunces', Georgia, serif);
  font-size: 110px;
  line-height: 0.5;
  color: var(--nu-tour);
  font-weight: 700;
  text-shadow: 0 4px 24px rgba(251, 217, 69, 0.4);
  margin-bottom: 18px;
}
.quote-body {
  font-family: var(--font-serif, 'Fraunces', Georgia, serif);
  font-size: 44px;
  line-height: 1.32;
  font-weight: 500;
  font-style: italic;
  color: var(--nu-wisp);
  margin: 0 0 16px 0;
  padding: 0;
  letter-spacing: 0.005em;
}
.quote-cite {
  font-size: 16px;
  letter-spacing: 0.32em;
  text-transform: uppercase;
  color: var(--nu-skylight);
  font-weight: 500;
}

/* ============================================================
 * Footer glassmorphism card
 * ============================================================ */
.footer {
  position: absolute;
  bottom: 64px;
  left: 0;
  right: 0;
  z-index: 3;
  display: flex;
  justify-content: center;
  padding: 0 64px;
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.9s ease, transform 0.9s ease;
}
.footer.is-in {
  opacity: 1;
  transform: translateY(0);
}

.footer-card {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 28px;
  padding: 22px 36px 22px 30px;
  background: rgba(0, 38, 61, 0.45);
  border: 1px solid rgba(251, 217, 69, 0.28);
  border-radius: 24px;
  backdrop-filter: blur(20px) saturate(140%);
  -webkit-backdrop-filter: blur(20px) saturate(140%);
  box-shadow:
    0 18px 56px rgba(0, 0, 0, 0.42),
    inset 0 1px 0 rgba(255, 255, 255, 0.08);
  max-width: 1080px;
}

.footer-accent {
  position: absolute;
  left: -1px;
  top: 18px;
  bottom: 18px;
  width: 3px;
  background: linear-gradient(180deg, transparent 0%, var(--nu-tour) 50%, transparent 100%);
  border-radius: 2px;
  box-shadow: 0 0 12px rgba(251, 217, 69, 0.6);
}

.footer-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
  line-height: 1.2;
}
.footer-line {
  font-size: 20px;
  color: var(--nu-skylight);
  font-weight: 500;
  letter-spacing: 0.01em;
}
.footer-line--strong {
  color: var(--nu-wisp);
  font-weight: 600;
}

.footer-meta {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding-left: 28px;
  margin-left: 4px;
  border-left: 1px solid rgba(251, 217, 69, 0.25);
  font-size: 14px;
  letter-spacing: 0.28em;
  text-transform: uppercase;
  color: var(--nu-tour);
  font-weight: 600;
}
.meta-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--nu-tour);
  box-shadow: 0 0 10px rgba(251, 217, 69, 0.8);
  animation: pulse 2.4s ease-in-out infinite;
}
@keyframes pulse {
  0%, 100% { opacity: 0.6; transform: scale(1); }
  50%      { opacity: 1;   transform: scale(1.15); }
}

/* ============================================================
 * Edge hairline at the very bottom for frame finish.
 * ============================================================ */
.edge-rule {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent 0%, var(--nu-tour) 35%, var(--nu-amber) 65%, transparent 100%);
  z-index: 4;
  opacity: 0.7;
}
</style>
