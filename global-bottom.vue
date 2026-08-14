<!--
  Wave motif rendered behind every slide.
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
</template>

<script setup lang="ts">
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
  fill: rgba(94, 205, 255, 0.13);
  animation: drift 26s linear infinite;
}

.wave-mid {
  height: 52%;
  fill: rgba(63, 224, 240, 0.16);
  animation: drift 18s linear infinite reverse;
}

.wave-front {
  height: 34%;
  fill: rgba(165, 227, 255, 0.20);
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

/* Freeze the motion for print/PDF export and for reduced-motion users */
@media print, (prefers-reduced-motion: reduce) {
  .wave {
    animation: none !important;
  }
}
</style>
