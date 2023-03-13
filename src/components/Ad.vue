<template>
  <div v-if="!selfKey && $window.show_ads" class="mt-3">
    <span class="rounded px-1 py-0.5 border text-xs font-bold bg-yellow-400 border-yellow-400
text-gray-800">Ad</span>
    <span class="px-1 font-bold"> ·</span>
    <a :href="$window.promot_link" target="_blank" class="text-green-700 hover:text-green-600
transition">{{ $window.promot_text }}</a>
  </div>
</template>

<script setup>
  const selfKey = localStorage.getItem('api_key')
</script>
