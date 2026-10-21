<script setup>
import { onMounted, onUnmounted } from 'vue'

function openExternalInNewTab(e) {
  const a = e.target.closest && e.target.closest('a[href]')
  if (!a || !/^https?:/i.test(a.href) || a.target === '_blank') return
  e.preventDefault()
  window.open(a.href, '_blank', 'noopener,noreferrer')
}

onMounted(() => document.addEventListener('click', openExternalInNewTab, true))
onUnmounted(() => document.removeEventListener('click', openExternalInNewTab, true))
</script>

<template>
  <div class="slide-number">{{ $nav.currentPage }} / {{ $nav.total }}</div>
</template>

<style>
.slide-number {
  position: absolute;
  right: 1.5rem;
  bottom: 0.8rem;
  z-index: 100;
  font-size: 1rem;
  color: #c9d3dc;
}
</style>
