<template>
  <div class="min-h-screen bg-zinc-950 text-zinc-100 flex flex-col justify-between selection:bg-zinc-800 selection:text-zinc-100">
    <ColorPaletteExtractor />
    
    <!-- Minimalist footer -->
    <footer class="w-full border-t border-zinc-900 py-6 px-4 text-center text-xs text-zinc-400 font-mono">
      <span>Chroma &copy; 2026. Non-blocking client-side color extraction.</span>
    </footer>
  </div>
</template>

<script setup>
import ColorPaletteExtractor from './components/ColorPaletteExtractor.vue';
</script>
