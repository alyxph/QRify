<template>
  <nav class="fixed w-full z-30 top-4 left-0">
    <div class="max-w-screen-xl mx-auto px-4">
      <div class="bg-white/70 backdrop-blur-md border border-white/30 rounded-xl shadow-lg py-3 px-5 flex items-center justify-center">
        <a href="/" class="flex items-center gap-3">
          <!-- site logo (three-square QR-like mark) -->
          <svg width="48" height="48" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" class="text-sky-600">
            <rect x="2" y="2" width="7" height="7" rx="1" fill="currentColor" />
            <rect x="15" y="2" width="7" height="7" rx="1" fill="currentColor" />
            <rect x="2" y="15" width="7" height="7" rx="1" fill="currentColor" />
          </svg>
          <div class="text-center">
            <div class="text-lg font-bold text-slate-800">QRify</div>
            <div class="text-xs text-slate-500 -mt-0.5">Simple QR generator</div>
          </div>
        </a>
      </div>
    </div>
  </nav>
</template>