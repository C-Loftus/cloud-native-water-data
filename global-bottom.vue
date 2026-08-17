<!--
  Wave motif rendered behind every slide, plus the CGS logo lockup pinned to the
  bottom-right corner of every slide.
  Three stacked SVG bands drifting at different speeds to suggest moving water.
-->
<template>
  <div class="water-waves" aria-hidden="true">
    <svg class="wave wave-back" viewBox="0 0 1440 180" preserveAspectRatio="none">
      <path :d="wavePath" />
    </svg>
    <svg class="wave wave-mid" viewBox="0 0 1440 180" preserveAspectRatio="none">
      <path :d="wavePath" />
    </svg>
    <svg class="wave wave-front" viewBox="0 0 1440 180" preserveAspectRatio="none">
      <path :d="wavePath" />
    </svg>
  </div>

  <div v-if="showMark" class="cgs-mark">
    <img src="/cgs-logo-white.png" alt="The Center for Geospatial Solutions" />
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useNav } from '@slidev/client'

// A slide opts out of the corner lockup with `hideLogo: true` in its frontmatter.
// The live-demo slide does, so the mark never sits on top of the embedded viewer.
const { currentSlideRoute } = useNav()
const showMark = computed(
  () => currentSlideRoute.value?.meta?.slide?.frontmatter?.hideLogo !== true,
)

// One tiling period is 720 units wide, drawn twice so the loop is seamless.
const wavePath = [
  'M0 90',
  'C 120 40, 240 40, 360 90',
  'S 600 140, 720 90',
  'S 960 40, 1080 90',
  'S 1320 140, 1440 90',
  'L1440 180 L0 180 Z',
].join(' ')
</script>

<style scoped>
.water-waves {
  position: absolute;
  inset: auto 0 0 0;
  height: 46%;
  pointer-events: none;
  overflow: hidden;
  z-index: 0;
}

/* The path repeats every 720 viewBox units. Stretched across 200% width that is
   exactly one slide width, so translating by 50% of the element loops seamlessly.
   Starting at left:-100% keeps the slide covered at both ends of the cycle. */
.wave {
  position: absolute;
  bottom: 0;
  left: -100%;
  width: 200%;
  will-change: transform;
}

.wave-back {
  height: 68%;
  fill: rgba(44, 132, 185, 0.30);
  animation: drift 26s linear infinite;
}

.wave-mid {
  height: 52%;
  fill: rgba(70, 171, 157, 0.20);
  animation: drift 18s linear infinite reverse;
}

.wave-front {
  height: 34%;
  fill: rgba(201, 222, 232, 0.20);
  animation: drift 12s linear infinite;
}

@keyframes drift {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(50%);
  }
}

/* Sits above the slide layout so it survives full-bleed slides (the iframe demo).
   The all-white lockup is the highest-contrast option against the dark blue stage. */
.cgs-mark {
  position: absolute;
  right: 1.15rem;
  bottom: 0.9rem;
  z-index: 20;
  pointer-events: none;
  line-height: 0;
}

.cgs-mark img {
  height: 2.1rem;
  width: auto;
}

/* Freeze the motion for print/PDF export and for reduced-motion users */
@media print, (prefers-reduced-motion: reduce) {
  .wave {
    animation: none !important;
  }
}
</style>
