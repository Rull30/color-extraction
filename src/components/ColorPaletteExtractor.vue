<template>
  <div class="w-full max-w-6xl mx-auto px-4 py-8 sm:py-12 flex flex-col gap-8">
    <!-- Header: Linear / Raycast inspired minimal branding -->
    <header class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 pb-6 border-b border-zinc-800/80">
      <div class="space-y-1">
        <div class="flex items-center gap-2.5">
          <div class="w-2.5 h-2.5 rounded-sm bg-zinc-300 shadow-[0_0_8px_rgba(255,255,255,0.2)]"></div>
          <h1 class="text-xl sm:text-2xl font-semibold tracking-tight text-zinc-100">
            Chroma Extract
          </h1>
          <span class="text-[11px] font-medium tracking-wide uppercase px-2 py-0.5 rounded border border-zinc-800 bg-zinc-900 text-zinc-400">
            Palette Tool
          </span>
        </div>
        <p class="text-xs sm:text-sm text-zinc-400 font-normal">
          Ekstrak palet warna esensial dan harmonis dari gambar secara instan langsung di peramban.
        </p>
      </div>

      <!-- Action toolbar: Format toggle & Palette size -->
      <div class="flex items-center gap-2.5 self-start sm:self-auto">
        <!-- Color format selector -->
        <div class="inline-flex p-1 rounded-lg bg-zinc-900 border border-zinc-800 text-xs" role="group" aria-label="Format warna">
          <button
            v-for="fmt in formats"
            :key="fmt"
            type="button"
            @click="activeFormat = fmt"
            :class="[
              'px-2.5 py-1 rounded-md font-mono text-[11px] transition-colors',
              activeFormat === fmt
                ? 'bg-zinc-800 text-zinc-100 font-medium shadow-sm'
                : 'text-zinc-400 hover:text-zinc-200'
            ]"
            :aria-pressed="activeFormat === fmt"
          >
            {{ fmt }}
          </button>
        </div>

        <!-- Palette count selector -->
        <div class="flex items-center gap-1.5 px-2.5 py-1 rounded-lg bg-zinc-900 border border-zinc-800 text-xs text-zinc-400">
          <label for="color-count-select" class="text-[11px] text-zinc-400 select-none">Warna:</label>
          <select
            id="color-count-select"
            v-model.number="colorCount"
            @change="handleColorCountChange"
            class="bg-transparent text-zinc-200 text-xs font-mono font-medium focus:outline-none cursor-pointer"
          >
            <option :value="5" class="bg-zinc-900 text-zinc-200">5</option>
            <option :value="6" class="bg-zinc-900 text-zinc-200">6</option>
            <option :value="8" class="bg-zinc-900 text-zinc-200">8</option>
            <option :value="10" class="bg-zinc-900 text-zinc-200">10</option>
          </select>
        </div>
      </div>
    </header>

    <!-- Main Workspace -->
    <main class="flex flex-col gap-8">
      <!-- Hidden native file input -->
      <input
        ref="fileInputRef"
        type="file"
        accept="image/png,image/jpeg,image/webp,image/svg+xml,image/avif"
        class="hidden"
        @change="handleFileSelect"
      />

      <!-- Dropzone Section: shown when no image or in compact mode when image exists -->
      <section v-if="!imageSrc" aria-labelledby="dropzone-heading">
        <h2 id="dropzone-heading" class="sr-only">Unggah atau seret gambar</h2>
        
        <div
          @dragover.prevent="isDragging = true"
          @dragleave.prevent="isDragging = false"
          @drop.prevent="handleDrop"
          @click="triggerFileInput"
          @keydown.enter="triggerFileInput"
          @keydown.space.prevent="triggerFileInput"
          tabindex="0"
          role="button"
          aria-label="Area unggah gambar, klik atau seret gambar ke sini"
          :class="[
            'relative group cursor-pointer rounded-2xl border-2 border-dashed transition-all duration-300 ease-out p-8 sm:p-12 flex flex-col items-center justify-center text-center overflow-hidden min-h-[260px]',
            isDragging
              ? 'border-zinc-400 bg-zinc-900/90 scale-[1.008] shadow-[0_0_30px_rgba(255,255,255,0.06)]'
              : 'border-zinc-800 bg-zinc-900/30 hover:border-zinc-700 hover:bg-zinc-900/50 hover:shadow-[0_0_25px_rgba(255,255,255,0.03)]'
          ]"
        >
          <!-- Subtle ambient light accent inside dropzone -->
          <div class="pointer-events-none absolute inset-0 bg-[radial-gradient(ellipse_60%_50%_at_50%_0%,rgba(255,255,255,0.03),transparent_70%)]"></div>

          <!-- Loading state overlay inside dropzone -->
          <div v-if="isLoading" class="relative z-10 flex flex-col items-center gap-3">
            <div class="w-7 h-7 border-2 border-zinc-700 border-t-zinc-200 rounded-full animate-spin"></div>
            <p class="text-xs font-mono text-zinc-300">Menganalisis kuantisasi warna gambar...</p>
          </div>

          <!-- Normal Dropzone content -->
          <div v-else class="relative z-10 flex flex-col items-center gap-3.5">
            <div class="w-12 h-12 rounded-xl border border-zinc-800 bg-zinc-900 flex items-center justify-center text-zinc-400 group-hover:text-zinc-200 group-hover:border-zinc-700 transition-colors">
              <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
                <path stroke-linecap="round" stroke-linejoin="round" d="M2.25 15.75l5.159-5.159a2.25 2.25 0 013.182 0l5.159 5.159m-1.5-1.5l1.409-1.409a2.25 2.25 0 013.182 0l2.909 2.909m-18 3.75h16.5a1.5 1.5 0 001.5-1.5V6a1.5 1.5 0 00-1.5-1.5H3.75A1.5 1.5 0 002.25 6v12a1.5 1.5 0 001.5 1.5zm10.5-11.25h.008v.008h-.008V8.25zm.375 0a.375.375 0 11-.75 0 .375.375 0 01.75 0z" />
              </svg>
            </div>

            <div class="space-y-1">
              <p class="text-sm font-medium text-zinc-200">
                <span class="text-zinc-100 underline decoration-zinc-700 underline-offset-4 group-hover:decoration-zinc-400 transition-colors">Pilih file gambar</span> atau seret dan lepas ke area ini
              </p>
              <p class="text-xs text-zinc-500">
                Mendukung format PNG, JPG, WebP, SVG, atau tempel langsung dengan <kbd class="px-1.5 py-0.5 rounded border border-zinc-800 bg-zinc-900 font-mono text-[10px] text-zinc-400">Ctrl + V</kbd>
              </p>
            </div>

            <!-- Quick try sample button -->
            <button
              type="button"
              @click.stop="loadSampleImage"
              class="mt-2 inline-flex items-center gap-1.5 px-3.5 py-1.5 rounded-lg border border-zinc-800 bg-zinc-900 text-xs font-medium text-zinc-300 hover:text-zinc-100 hover:border-zinc-700 hover:bg-zinc-850 transition-all focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none"
            >
              <svg class="w-3.5 h-3.5 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M9.813 15.904L9 18.75l-.813-2.846a4.5 4.5 0 00-3.09-3.09L2.25 12l2.846-.813a4.5 4.5 0 003.09-3.09L9 5.25l.813 2.846a4.5 4.5 0 003.09 3.09L15.75 12l-2.846.813a4.5 4.5 0 00-3.09 3.09zM18.259 8.715L18 9.75l-.259-1.035a3.375 3.375 0 00-2.455-2.456L14.25 6l1.036-.259a3.375 3.375 0 002.455-2.456L18 2.25l.259 1.035a3.375 3.375 0 002.456 2.456L21.75 6l-1.035.259a3.375 3.375 0 00-2.456 2.456z" />
              </svg>
              Gunakan Contoh Gambar
            </button>
          </div>
        </div>
      </section>

      <!-- Status indicator: Error message banner -->
      <div v-if="errorMessage" class="p-4 rounded-xl border border-red-900/60 bg-red-950/30 text-red-300 text-xs flex items-center justify-between gap-3" role="alert">
        <div class="flex items-center gap-2.5">
          <svg class="w-4 h-4 text-red-400 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m9-.75a9 9 0 11-18 0 9 9 0 0118 0zm-9 3.75h.008v.008H12v-.008z" />
          </svg>
          <span>{{ errorMessage }}</span>
        </div>
        <button
          type="button"
          @click="errorMessage = ''"
          class="text-red-400 hover:text-red-200 font-medium px-2 py-1 rounded transition-colors"
        >
          Tutup
        </button>
      </div>

      <!-- Active Content: Image Preview & Extracted Palette (Main Stable Workspace) -->
      <section v-if="imageSrc" class="flex flex-col gap-6" aria-label="Hasil ekstraksi palet warna">
        <!-- Control bar above results -->
        <div class="flex flex-wrap items-center justify-between gap-3">
          <div class="flex items-center gap-2">
            <span class="text-xs font-medium text-zinc-300">Hasil Analisis</span>
            <span class="text-[11px] text-zinc-500 font-mono">({{ paletteColors.length }} nada terdeteksi)</span>
            <!-- Small non-intrusive loading indicator when recalculating -->
            <span v-if="isLoading" class="inline-flex items-center gap-1.5 text-[11px] font-mono text-zinc-400 bg-zinc-900 border border-zinc-800 px-2 py-0.5 rounded-full">
              <span class="w-2 h-2 border border-zinc-400 border-t-transparent rounded-full animate-spin"></span>
              Menghitung ulang...
            </span>
          </div>

          <div class="flex items-center gap-2">
            <button
              type="button"
              @click="triggerFileInput"
              class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg border border-zinc-800 bg-zinc-900 text-xs font-medium text-zinc-300 hover:text-zinc-100 hover:border-zinc-700 transition-colors focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none"
            >
              <svg class="w-3.5 h-3.5 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 005.25 21h13.5A2.25 2.25 0 0021 18.75V16.5m-13.5-9L12 3m0 0l4.5 4.5M12 3v13.5" />
              </svg>
              Ganti Gambar
            </button>

            <!-- Export CSS Variables -->
            <button
              type="button"
              @click="exportCssVariables"
              class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg border border-zinc-800 bg-zinc-900 text-xs font-medium text-zinc-300 hover:text-zinc-100 hover:border-zinc-700 transition-colors focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none"
              title="Salin semua warna sebagai variabel CSS"
            >
              <svg class="w-3.5 h-3.5 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M17.25 6.75L22.5 12l-5.25 5.25m-10.5 0L1.5 12l5.25-5.25m7.5-3l-4.5 16.5" />
              </svg>
              Salin CSS
            </button>

            <!-- Export JSON -->
            <button
              type="button"
              @click="exportJsonArray"
              class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg border border-zinc-800 bg-zinc-900 text-xs font-medium text-zinc-300 hover:text-zinc-100 hover:border-zinc-700 transition-colors focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none"
              title="Salin warna dalam format JSON array"
            >
              <svg class="w-3.5 h-3.5 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 7.5l.415-.207a.75.75 0 011.085.67V10.5m0 0h6m-6 0h-1.5m7.5 0v2.53a.75.75 0 01-1.085.67L14.25 13.5M6 18h12a2.25 2.25 0 002.25-2.25V7.5A2.25 2.25 0 0018 5.25H6A2.25 2.25 0 003.75 7.5v8.25A2.25 2.25 0 006 18z" />
              </svg>
              Salin JSON
            </button>

            <!-- Reset workspace -->
            <button
              type="button"
              @click="resetWorkspace"
              class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg border border-zinc-800 bg-zinc-900/60 text-xs font-medium text-zinc-400 hover:text-zinc-200 hover:border-zinc-700 transition-colors focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none"
            >
              Reset
            </button>
          </div>
        </div>

        <!-- Two column layout: Preview and Dominant card on left, Palette swatches on right -->
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
          <!-- Left column: Source Image Preview and Dominant Color Hero -->
          <div class="lg:col-span-5 flex flex-col gap-4">
            <!-- Source Image Card -->
            <div class="rounded-xl border border-zinc-800 bg-zinc-900/60 overflow-hidden">
              <div class="px-3.5 py-2.5 border-b border-zinc-800/80 flex items-center justify-between text-xs">
                <span class="text-zinc-400 font-medium">Gambar Sumber</span>
                <span class="text-[11px] font-mono text-zinc-400 truncate max-w-[180px]">{{ currentFileName }}</span>
              </div>
              <div class="relative max-h-64 sm:max-h-80 overflow-hidden bg-zinc-950 flex items-center justify-center p-2">
                <img
                  :src="imageSrc"
                  :alt="'Pratinjau gambar ' + currentFileName"
                  class="w-full h-auto max-h-72 object-contain rounded-lg"
                />
              </div>
            </div>

            <!-- Dominant Hero Swatch Card -->
            <div
              v-if="dominantColor"
              class="rounded-xl border border-zinc-800 bg-zinc-900/80 p-4 flex flex-col gap-3 transition-all duration-200 hover:border-zinc-700"
            >
              <div class="flex items-center justify-between">
                <div class="flex items-center gap-2">
                  <span class="text-xs font-medium text-zinc-200">Warna Dominan</span>
                  <span class="text-[10px] uppercase font-mono px-1.5 py-0.5 rounded bg-zinc-800 text-zinc-400 border border-zinc-700">
                    Primary
                  </span>
                </div>
                <span class="text-[11px] text-zinc-400 font-mono">
                  {{ dominantColor.isLight ? 'Nada Terang' : 'Nada Gelap' }}
                </span>
              </div>

              <div class="flex items-stretch gap-3">
                <!-- Color block -->
                <div
                  class="w-16 h-16 rounded-lg border border-black/10 shrink-0 shadow-inner transition-transform hover:scale-105 duration-200 cursor-pointer"
                  :style="{ backgroundColor: dominantColor.hex }"
                  @click="copyColor(dominantColor, 'HEX')"
                  title="Klik untuk menyalin HEX"
                  role="button"
                  tabindex="0"
                  @keydown.enter="copyColor(dominantColor, 'HEX')"
                ></div>

                <!-- Info and copy action -->
                <div class="flex-1 flex flex-col justify-center gap-1 min-w-0">
                  <div class="flex items-center justify-between">
                    <span class="font-mono text-sm font-semibold text-zinc-100 tracking-wide">
                      {{ formatColor(dominantColor) }}
                    </span>
                    <button
                      type="button"
                      @click="copyColor(dominantColor, activeFormat)"
                      class="px-2 py-1 rounded bg-zinc-800/80 hover:bg-zinc-700 text-zinc-200 hover:text-white text-xs font-mono transition-colors focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none"
                      :aria-label="'Salin warna dominan ' + dominantColor.hex"
                    >
                      Salin
                    </button>
                  </div>
                  <p class="font-mono text-[11px] text-zinc-400 truncate">
                    RGB: {{ dominantColor.rgbString }}
                  </p>
                </div>
              </div>
            </div>
          </div>

          <!-- Right column: Extracted Palette Grid -->
          <div class="lg:col-span-7 flex flex-col gap-4">
            <div class="flex items-center justify-between">
              <h3 class="text-xs font-medium text-zinc-300">Variasi Palet Harmonis</h3>
              <p class="text-[11px] text-zinc-400">Klik pada kartu untuk menyalin nilai warna</p>
            </div>

            <!-- Swatches Grid with micro-interactions -->
            <div
              :class="[
                'grid grid-cols-2 sm:grid-cols-3 gap-3.5 transition-opacity duration-200',
                isLoading ? 'opacity-60 pointer-events-none' : 'opacity-100'
              ]"
            >
              <div
                v-for="(color, index) in paletteColors"
                :key="color.hex + '-' + index"
                @click="copyColor(color, activeFormat)"
                @keydown.enter="copyColor(color, activeFormat)"
                @keydown.space.prevent="copyColor(color, activeFormat)"
                tabindex="0"
                role="button"
                :aria-label="`Warna ${index + 1}: ${color.hex}, klik untuk menyalin`"
                class="group rounded-xl border border-zinc-800/90 bg-zinc-900/70 overflow-hidden cursor-pointer hover:border-zinc-700 hover:-translate-y-1 hover:shadow-lg transition-all duration-200 focus-visible:ring-2 focus-visible:ring-zinc-400 focus-visible:outline-none flex flex-col"
              >
                <!-- Color Display Block -->
                <div
                  class="h-24 w-full relative transition-transform duration-200 group-hover:scale-[1.01]"
                  :style="{ backgroundColor: color.hex }"
                >
                  <!-- Subtle corner tag index -->
                  <span
                    :class="[
                      'absolute top-2 left-2 text-[10px] font-mono font-medium px-1.5 py-0.5 rounded backdrop-blur-sm transition-opacity opacity-70 group-hover:opacity-100',
                      color.isLight ? 'bg-black/20 text-zinc-900 border border-black/10' : 'bg-white/20 text-zinc-100 border border-white/10'
                    ]"
                  >
                    0{{ index + 1 }}
                  </span>

                  <!-- Hover copy badge -->
                  <div class="absolute inset-0 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity bg-black/15">
                    <span
                      :class="[
                        'text-[10px] font-mono font-medium px-2 py-1 rounded shadow-sm backdrop-blur-md',
                        color.isLight ? 'bg-zinc-900 text-zinc-100' : 'bg-zinc-100 text-zinc-900'
                      ]"
                    >
                      Salin {{ activeFormat }}
                    </span>
                  </div>
                </div>

                <!-- Color Metadata and Quick Action -->
                <div class="p-3 bg-zinc-900/90 border-t border-zinc-800/80 flex flex-col gap-1 text-left">
                  <div class="flex items-center justify-between">
                    <span class="font-mono text-xs font-semibold text-zinc-200 tracking-wide">
                      {{ formatColor(color) }}
                    </span>
                    <button
                      type="button"
                      @click.stop="copyColor(color, activeFormat)"
                      class="p-1 rounded text-zinc-400 hover:text-zinc-100 hover:bg-zinc-800 transition-colors focus-visible:ring-1 focus-visible:ring-zinc-400"
                      :title="'Salin ' + color.hex"
                      :aria-label="'Salin warna ' + color.hex"
                    >
                      <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M15.666 3.888A2.25 2.25 0 0013.5 2.25h-3c-1.03 0-1.9.693-2.166 1.638m7.332 0c.055.194.084.4.084.612v0a.75.75 0 01-.75.75H9a.75.75 0 01-.75-.75v0c0-.212.03-.418.084-.612m7.332 0c.646.049 1.288.11 1.927.184 1.1.128 1.907 1.077 1.907 2.185V19.5a2.25 2.25 0 01-2.25 2.25H6.75A2.25 2.25 0 014.5 19.5V6.257c0-1.108.806-2.057 1.907-2.185a48.208 48.208 0 011.927-.184" />
                      </svg>
                    </button>
                  </div>
                  <span class="font-mono text-[10px] text-zinc-400 truncate">
                    {{ activeFormat === 'HEX' ? color.rgbString : color.hex }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- Toast Notification: Minimalist Floating Feedback -->
    <Teleport to="body">
      <Transition
        enter-active-class="transition duration-300 ease-out transform"
        enter-from-class="opacity-0 translate-y-3 scale-95"
        enter-to-class="opacity-100 translate-y-0 scale-100"
        leave-active-class="transition duration-200 ease-in transform"
        leave-from-class="opacity-100 translate-y-0 scale-100"
        leave-to-class="opacity-0 translate-y-3 scale-95"
      >
        <div
          v-if="toast.visible"
          role="status"
          aria-live="polite"
          class="fixed bottom-6 left-1/2 -translate-x-1/2 z-50 pointer-events-none"
        >
          <div class="flex items-center gap-2.5 px-3.5 py-2 rounded-full border border-zinc-800 bg-zinc-900/95 text-zinc-100 shadow-xl backdrop-blur-md text-xs font-mono">
            <span
              v-if="toast.colorHex"
              class="w-3 h-3 rounded-full border border-white/20 shrink-0 shadow-sm"
              :style="{ backgroundColor: toast.colorHex }"
            ></span>
            <svg class="w-3.5 h-3.5 text-emerald-400 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
              <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 12.75l6 6 9-13.5" />
            </svg>
            <span class="text-zinc-200 font-sans text-xs">{{ toast.message }}</span>
          </div>
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue';
import ColorThief from 'colorthief';

// ==========================================
// Types and Reactive States
// ==========================================
const formats = ['HEX', 'RGB', 'HSL'];
const activeFormat = ref('HEX');
const colorCount = ref(6);

const fileInputRef = ref(null);

const isDragging = ref(false);
const isLoading = ref(false);
const errorMessage = ref('');

const imageSrc = ref('');
const currentFileName = ref('');
const dominantColor = ref(null);
const paletteColors = ref([]);

// Keep the loaded in-memory image cached so changing color count is instant
let loadedImageCache = null;

// Toast Notification state
const toast = reactive({
  visible: false,
  message: '',
  colorHex: '',
  timer: null,
});

// ==========================================
// Color Helper Functions (Pure and Accessible)
// ==========================================
function rgbToHex([r, g, b]) {
  const toHex = (c) => Math.max(0, Math.min(255, Math.round(c))).toString(16).padStart(2, '0');
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`.toUpperCase();
}

function rgbToHsl([r, g, b]) {
  const rNorm = r / 255;
  const gNorm = g / 255;
  const bNorm = b / 255;
  const max = Math.max(rNorm, gNorm, bNorm);
  const min = Math.min(rNorm, gNorm, bNorm);
  let h = 0;
  let s = 0;
  const l = (max + min) / 2;

  if (max !== min) {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch (max) {
      case rNorm:
        h = (gNorm - bNorm) / d + (gNorm < bNorm ? 6 : 0);
        break;
      case gNorm:
        h = (bNorm - rNorm) / d + 2;
        break;
      case bNorm:
        h = (rNorm - gNorm) / d + 4;
        break;
    }
    h /= 6;
  }
  return `hsl(${Math.round(h * 360)}, ${Math.round(s * 100)}%, ${Math.round(l * 100)}%)`;
}

// Relative luminance based on WCAG standard to ensure accessible contrast markers
function calculateLuminance([r, g, b]) {
  const a = [r, g, b].map((v) => {
    const channel = v / 255;
    return channel <= 0.03928 ? channel / 12.92 : Math.pow((channel + 0.055) / 1.055, 2.4);
  });
  return a[0] * 0.2126 + a[1] * 0.7152 + a[2] * 0.0722;
}

function createColorObject(rgb) {
  const hex = rgbToHex(rgb);
  const hsl = rgbToHsl(rgb);
  const isLight = calculateLuminance(rgb) > 0.45;
  return {
    rgb,
    rgbString: `rgb(${rgb[0]}, ${rgb[1]}, ${rgb[2]})`,
    hex,
    hsl,
    isLight,
  };
}

function formatColor(colorObj) {
  if (!colorObj) return '';
  if (activeFormat.value === 'HEX') return colorObj.hex;
  if (activeFormat.value === 'RGB') return colorObj.rgbString;
  if (activeFormat.value === 'HSL') return colorObj.hsl;
  return colorObj.hex;
}

// ==========================================
// Toast Handler
// ==========================================
function showToast(message, colorHex = '') {
  if (toast.timer) clearTimeout(toast.timer);
  toast.message = message;
  toast.colorHex = colorHex;
  toast.visible = true;

  toast.timer = setTimeout(() => {
    toast.visible = false;
  }, 2400);
}

// ==========================================
// Copy to Clipboard Actions
// ==========================================
async function copyColor(colorObj, format) {
  if (!colorObj) return;
  let textToCopy = colorObj.hex;
  if (format === 'RGB') textToCopy = colorObj.rgbString;
  else if (format === 'HSL') textToCopy = colorObj.hsl;

  try {
    await navigator.clipboard.writeText(textToCopy);
    showToast(`Berhasil menyalin ${textToCopy}`, colorObj.hex);
  } catch {
    showToast(`Disalin: ${textToCopy}`, colorObj.hex);
  }
}

async function exportCssVariables() {
  if (!paletteColors.value.length) return;
  const lines = [':root {'];
  if (dominantColor.value) {
    lines.push(`  --color-dominant: ${dominantColor.value.hex};`);
  }
  paletteColors.value.forEach((color, idx) => {
    lines.push(`  --color-${idx + 1}: ${color.hex};`);
  });
  lines.push('}');
  const css = lines.join('\n');

  try {
    await navigator.clipboard.writeText(css);
    showToast('Variabel CSS tersalin ke clipboard');
  } catch {
    showToast('Gagal menyalin CSS');
  }
}

async function exportJsonArray() {
  if (!paletteColors.value.length) return;
  const list = paletteColors.value.map((c) => c.hex);
  const json = JSON.stringify(list, null, 2);

  try {
    await navigator.clipboard.writeText(json);
    showToast('JSON palet tersalin ke clipboard');
  } catch {
    showToast('Gagal menyalin JSON');
  }
}

// ==========================================
// Resilient Color Extraction with ColorThief & Canvas Fallback
// ==========================================
function extractColorsFromCanvas(canvas, targetCount) {
  // Strategy 1: ColorThief
  try {
    const ColorThiefConstructor = ColorThief?.default || ColorThief || (typeof window !== 'undefined' ? window.ColorThief : null);
    const thief = typeof ColorThiefConstructor === 'function' ? new ColorThiefConstructor() : ColorThiefConstructor;

    if (thief && typeof thief.getPalette === 'function') {
      const dominantRgb = thief.getColor(canvas, 10);
      const rawPalette = thief.getPalette(canvas, Math.max(targetCount, 4), 10);

      if (rawPalette && rawPalette.length > 0) {
        return {
          dominant: dominantRgb || rawPalette[0],
          palette: rawPalette.slice(0, targetCount),
        };
      }
    }
  } catch (thiefErr) {
    console.warn('ColorThief notice:', thiefErr);
  }

  // Strategy 2: High-speed canvas pixel quantization fallback
  const ctx = canvas.getContext('2d');
  const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const data = imgData.data;
  const colorBuckets = new Map();
  const step = 4 * 4; // Sample every 4th pixel for speed

  for (let i = 0; i < data.length; i += step) {
    const a = data[i + 3];
    if (a < 125) continue;
    const r = data[i];
    const g = data[i + 1];
    const b = data[i + 2];

    const qr = Math.round(r / 10) * 10;
    const qg = Math.round(g / 10) * 10;
    const qb = Math.round(b / 10) * 10;
    const key = `${qr},${qg},${qb}`;

    const existing = colorBuckets.get(key) || { r: 0, g: 0, b: 0, count: 0 };
    existing.r += r;
    existing.g += g;
    existing.b += b;
    existing.count += 1;
    colorBuckets.set(key, existing);
  }

  const sorted = Array.from(colorBuckets.values()).sort((a, b) => b.count - a.count);

  if (!sorted.length) {
    return {
      dominant: [39, 39, 42],
      palette: [[24, 24, 27], [39, 39, 42], [63, 63, 70], [82, 82, 91], [113, 113, 122]],
    };
  }

  const palette = [];
  const minDistanceSq = 18 * 18 * 3;

  for (const item of sorted) {
    const avgR = Math.round(item.r / item.count);
    const avgG = Math.round(item.g / item.count);
    const avgB = Math.round(item.b / item.count);

    const isDistinct = palette.every(([pr, pg, pb]) => {
      const distSq = (avgR - pr) ** 2 + (avgG - pg) ** 2 + (avgB - pb) ** 2;
      return distSq >= minDistanceSq;
    });

    if (isDistinct || palette.length < 2) {
      palette.push([avgR, avgG, avgB]);
      if (palette.length >= targetCount) break;
    }
  }

  while (palette.length < targetCount && sorted[palette.length]) {
    const item = sorted[palette.length];
    palette.push([
      Math.round(item.r / item.count),
      Math.round(item.g / item.count),
      Math.round(item.b / item.count),
    ]);
  }

  const dominant = palette[0] || [
    Math.round(sorted[0].r / sorted[0].count),
    Math.round(sorted[0].g / sorted[0].count),
    Math.round(sorted[0].b / sorted[0].count),
  ];

  return { dominant, palette };
}

// In-memory extraction without DOM dependency
async function extractPaletteFromImageObject(imgElement) {
  // Create an offscreen canvas downsampled to max 300px
  const offscreenCanvas = document.createElement('canvas');
  const ctx = offscreenCanvas.getContext('2d');
  const maxDim = 300;
  let w = imgElement.naturalWidth || imgElement.width || 300;
  let h = imgElement.naturalHeight || imgElement.height || 300;

  if (w > maxDim || h > maxDim) {
    if (w > h) {
      h = Math.round((h * maxDim) / w);
      w = maxDim;
    } else {
      w = Math.round((w * maxDim) / h);
      h = maxDim;
    }
  }

  offscreenCanvas.width = Math.max(1, w);
  offscreenCanvas.height = Math.max(1, h);
  ctx.drawImage(imgElement, 0, 0, offscreenCanvas.width, offscreenCanvas.height);

  const { dominant, palette } = extractColorsFromCanvas(offscreenCanvas, colorCount.value);

  if (dominant) {
    dominantColor.value = createColorObject(dominant);
  }

  if (Array.isArray(palette) && palette.length > 0) {
    paletteColors.value = palette.map(createColorObject);
  }
}

// ==========================================
// Centralized Asynchronous Pipeline
// ==========================================
async function processImageSource(src, fileName) {
  isLoading.value = true;
  errorMessage.value = '';

  try {
    // Load image strictly in JavaScript memory first
    const inMemoryImg = new Image();
    inMemoryImg.crossOrigin = 'anonymous';

    await new Promise((resolve, reject) => {
      inMemoryImg.onload = () => resolve();
      inMemoryImg.onerror = () => reject(new Error('Gagal membaca gambar. Pastikan file valid.'));
      inMemoryImg.src = src;
    });

    // Cache the loaded image object for instant palette resizing
    loadedImageCache = inMemoryImg;

    // Run extraction
    await extractPaletteFromImageObject(inMemoryImg);

    // Update UI state only after colors are calculated
    currentFileName.value = fileName;
    imageSrc.value = src;
  } catch (err) {
    console.error(err);
    errorMessage.value = err.message || 'Tidak dapat memproses gambar ini. Harap pilih gambar lain.';
  } finally {
    isLoading.value = false;
  }
}

function processFile(file) {
  if (!file) return;

  if (!file.type.startsWith('image/')) {
    errorMessage.value = 'Format file tidak didukung. Harap pilih file gambar (PNG, JPG, WebP, SVG).';
    return;
  }

  errorMessage.value = '';

  // Revoke previous blob URL if exists
  if (imageSrc.value && imageSrc.value.startsWith('blob:')) {
    URL.revokeObjectURL(imageSrc.value);
  }

  const objectUrl = URL.createObjectURL(file);
  const name = file.name || 'Pasted-Image.png';
  processImageSource(objectUrl, name);
}

function triggerFileInput() {
  if (fileInputRef.value) {
    fileInputRef.value.click();
  }
}

function handleFileSelect(event) {
  const file = event.target.files?.[0];
  if (file) {
    processFile(file);
    event.target.value = '';
  }
}

function handleDrop(event) {
  isDragging.value = false;
  const file = event.dataTransfer?.files?.[0];
  if (file) {
    processFile(file);
  }
}

async function handleColorCountChange() {
  if (loadedImageCache) {
    isLoading.value = true;
    try {
      await extractPaletteFromImageObject(loadedImageCache);
    } finally {
      isLoading.value = false;
    }
  }
}

function resetWorkspace() {
  if (imageSrc.value && imageSrc.value.startsWith('blob:')) {
    URL.revokeObjectURL(imageSrc.value);
  }
  imageSrc.value = '';
  currentFileName.value = '';
  dominantColor.value = null;
  paletteColors.value = [];
  errorMessage.value = '';
  loadedImageCache = null;
}

// ==========================================
// Sample Image Generator
// ==========================================
function loadSampleImage() {
  const sampleCanvas = document.createElement('canvas');
  sampleCanvas.width = 800;
  sampleCanvas.height = 500;
  const ctx = sampleCanvas.getContext('2d');

  // Atmospheric twilight gradient
  const bgGradient = ctx.createLinearGradient(0, 0, 0, 500);
  bgGradient.addColorStop(0, '#0f172a');
  bgGradient.addColorStop(0.4, '#1e293b');
  bgGradient.addColorStop(0.7, '#334155');
  bgGradient.addColorStop(1, '#475569');
  ctx.fillStyle = bgGradient;
  ctx.fillRect(0, 0, 800, 500);

  // Soft atmospheric sun
  const sunGradient = ctx.createRadialGradient(400, 320, 10, 400, 320, 180);
  sunGradient.addColorStop(0, '#f59e0b');
  sunGradient.addColorStop(0.4, '#d97706');
  sunGradient.addColorStop(0.8, '#b45309');
  sunGradient.addColorStop(1, 'transparent');
  ctx.fillStyle = sunGradient;
  ctx.beginPath();
  ctx.arc(400, 320, 180, 0, Math.PI * 2);
  ctx.fill();

  // Mountain silhouettes with harmonious natural tones
  ctx.fillStyle = '#1e293b';
  ctx.beginPath();
  ctx.moveTo(0, 500);
  ctx.lineTo(0, 340);
  ctx.lineTo(160, 260);
  ctx.lineTo(310, 310);
  ctx.lineTo(490, 220);
  ctx.lineTo(650, 300);
  ctx.lineTo(800, 250);
  ctx.lineTo(800, 500);
  ctx.closePath();
  ctx.fill();

  ctx.fillStyle = '#0f172a';
  ctx.beginPath();
  ctx.moveTo(0, 500);
  ctx.lineTo(0, 390);
  ctx.lineTo(220, 330);
  ctx.lineTo(420, 410);
  ctx.lineTo(580, 320);
  ctx.lineTo(800, 380);
  ctx.lineTo(800, 500);
  ctx.closePath();
  ctx.fill();

  const dataUrl = sampleCanvas.toDataURL('image/png');
  processImageSource(dataUrl, 'Nordic-Twilight-Sample.png');
}

// ==========================================
// Clipboard Paste Event Listener (Ctrl + V)
// ==========================================
function handlePaste(event) {
  const items = event.clipboardData?.items;
  if (!items) return;

  for (let i = 0; i < items.length; i++) {
    if (items[i].type.startsWith('image/')) {
      const file = items[i].getAsFile();
      if (file) {
        processFile(file);
        break;
      }
    }
  }
}

onMounted(() => {
  window.addEventListener('paste', handlePaste);
});

onUnmounted(() => {
  window.removeEventListener('paste', handlePaste);
  if (imageSrc.value && imageSrc.value.startsWith('blob:')) {
    URL.revokeObjectURL(imageSrc.value);
  }
  if (toast.timer) clearTimeout(toast.timer);
  loadedImageCache = null;
});
</script>
