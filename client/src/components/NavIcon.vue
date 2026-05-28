<script setup>
defineProps({
  name: {
    type: String,
    required: true
  }
})
</script>

<template>
  <!-- grid: 4-square dashboard mark for Overview -->
  <svg v-if="name === 'grid'" width="18" height="18" viewBox="0 0 18 18" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
    <rect x="2" y="2" width="6" height="6" rx="1"/>
    <rect x="10" y="2" width="6" height="6" rx="1"/>
    <rect x="2" y="10" width="6" height="6" rx="1"/>
    <rect x="10" y="10" width="6" height="6" rx="1"/>
  </svg>

  <!-- box: cube outline for Inventory -->
  <svg v-else-if="name === 'box'" width="18" height="18" viewBox="0 0 18 18" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
    <path d="M9 2L16 5.5V12.5L9 16L2 12.5V5.5L9 2Z"/>
    <path d="M9 2L9 16"/>
    <path d="M2 5.5L16 5.5"/>
  </svg>

  <!-- receipt: document with lines for Orders -->
  <svg v-else-if="name === 'receipt'" width="18" height="18" viewBox="0 0 18 18" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
    <path d="M4 2H14V16L11.5 14.5L9 16L6.5 14.5L4 16V2Z"/>
    <line x1="7" y1="7" x2="11" y2="7"/>
    <line x1="7" y1="10" x2="11" y2="10"/>
  </svg>

  <!-- wallet: rectangle with interior detail for Finance -->
  <svg v-else-if="name === 'wallet'" width="18" height="18" viewBox="0 0 18 18" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
    <rect x="2" y="5" width="14" height="10" rx="1"/>
    <path d="M2 8H16"/>
    <path d="M5 2H13L16 5H2L5 2Z"/>
    <circle cx="13" cy="11.5" r="1" fill="currentColor" stroke="none"/>
  </svg>

  <!-- trend: upward line chart for Demand Forecast -->
  <svg v-else-if="name === 'trend'" width="18" height="18" viewBox="0 0 18 18" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
    <polyline points="2,14 6,9 10,11 16,4"/>
    <polyline points="12,4 16,4 16,8"/>
  </svg>

  <!-- doc: document with corner fold for Reports -->
  <svg v-else-if="name === 'doc'" width="18" height="18" viewBox="0 0 18 18" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
    <path d="M4 2H11L15 6V16H4V2Z"/>
    <path d="M11 2V6H15"/>
    <line x1="7" y1="9" x2="12" y2="9"/>
    <line x1="7" y1="12" x2="12" y2="12"/>
  </svg>
</template>
