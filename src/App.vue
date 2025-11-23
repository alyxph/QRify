
<template>
	<div class="min-h-screen bg-gradient-to-b from-sky-50 to-white text-slate-800">
		<Navbar />

		<main class="max-w-6xl mx-auto px-6 pb-12 pt-32 md:pt-36">
			<!-- Hero -->
			<section class="bg-gradient-to-r from-sky-500 to-indigo-600 text-white rounded-2xl p-8 shadow-xl mb-8">
				<div class="md:flex md:items-center md:justify-between gap-6">
					<div class="max-w-2xl">
						<h1 class="text-4xl md:text-5xl font-extrabold leading-tight">Buat QR Code dengan cepat dan gratis</h1>
						<p class="mt-4 text-lg text-sky-100/90">QRify membantu Anda membuat QR untuk link, teks, atau berbagi Wi‑Fi.</p>
						<div class="mt-6">
							<a href="#generator" class="inline-block bg-white text-sky-600 font-semibold px-5 py-3 rounded-lg shadow hover:opacity-95">Buat QR Sekarang</a>
						</div>
					</div>
					<div class="hidden md:block">
						<!-- subtle decorative QR mockup -->
						<div class="w-48 h-48 bg-white/20 rounded-lg flex items-center justify-center">
							<svg class="w-32 h-32 text-white/90" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="2" y="2" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.2"/><rect x="15" y="2" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.2"/><rect x="2" y="15" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.2"/></svg>
						</div>
					</div>
				</div>
			</section>

			<section class="grid grid-cols-1 md:grid-cols-2 gap-8">
				<!-- Left: Form & filters -->
				<div class="space-y-4">
					<div id="generator" class="bg-white rounded-2xl shadow-2xl p-6 border border-slate-100">
						<h2 class="text-2xl font-semibold mb-2">Buat QR</h2>

						<label class="block text-sm text-slate-600 mb-2">Tipe QR</label>
						<select v-model="qrType" class="w-full border rounded px-3 py-2 mb-4">
							<option value="text">Teks</option>
							<option value="url">Link / URL</option>
							<option value="wifi">Wi‑Fi</option>
						</select>

						<!-- Conditional inputs -->
						<div v-if="qrType === 'text'">
							<label class="block text-sm font-medium text-slate-700 mb-2">Teks</label>
							<textarea v-model="textInput" rows="4" class="w-full border rounded px-3 py-2 mb-3"></textarea>
						</div>

						<div v-if="qrType === 'url'">
							<label class="block text-sm font-medium text-slate-700 mb-2">URL</label>
							<input v-model="textInput" placeholder="https://example.com" class="w-full border rounded px-3 py-2 mb-3" />
						</div>

						<div v-if="qrType === 'wifi'" class="space-y-2">
							<label class="block text-sm font-medium text-slate-700">SSID</label>
							<input v-model="wifi.ssid" class="w-full border rounded px-3 py-2" />

							<label class="block text-sm font-medium text-slate-700">Password</label>
							<input v-model="wifi.password" type="password" class="w-full border rounded px-3 py-2" />

							<label class="block text-sm font-medium text-slate-700">Encryption</label>
							<select v-model="wifi.encryption" class="w-full border rounded px-3 py-2">
								<option value="WPA">WPA/WPA2</option>
								<option value="WEP">WEP</option>
								<option value="nopass">None</option>
							</select>
						</div>

						<div class="mt-4">
								<button @click="clear" :disabled="loading" class="bg-gray-100 px-3 py-2 rounded disabled:opacity-60">Clear</button>
						</div>
					</div>
				</div>

				<!-- Right: Preview & actions -->
				<div>
					<div class="bg-white rounded-lg shadow p-6 flex flex-col items-center">
						<h3 class="text-lg font-medium mb-4">Preview</h3>
						<div class="relative w-full">
							<div class="flex items-center justify-center border border-dashed border-slate-200 rounded-lg p-6 mb-4 w-full bg-slate-50">
								<canvas ref="canvas" :width="size" :height="size"></canvas>
							</div>
							<div v-if="loading" class="absolute inset-0 flex items-center justify-center bg-white/60 rounded-lg">
								<div class="inline-flex items-center gap-3 bg-white/80 backdrop-blur-sm px-4 py-3 rounded-lg shadow">
									<svg class="w-6 h-6 text-sky-600 animate-spin" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
										<circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="2" stroke-opacity="0.25" />
										<path d="M22 12a10 10 0 00-10-10" stroke="currentColor" stroke-width="2" stroke-linecap="round" />
									</svg>
									<span class="font-medium text-slate-700">Mempersiapkan QR...</span>
								</div>
							</div>
						</div>

						<div class="flex gap-3">
							<button @click="download" class="bg-green-600 text-white px-4 py-2 rounded">Download PNG</button>
						</div>

						<p v-if="error" class="mt-3 text-sm text-red-600">Error: {{ error }}</p>
					</div>
				</div>
			</section>
		</main>

		<Footer />
	</div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import QRCode from 'qrcode'
import Navbar from './components/navbar.vue'
import Footer from './components/footer.vue'

const qrType = ref('text')
const textInput = ref('Hello, QR!')
// keep an internal size but remove UI controls for it
const size = ref(256)
const canvas = ref(null)
const error = ref(null)

const wifi = ref({ ssid: '', password: '', encryption: 'WPA' })

function buildPayload() {
	if (qrType.value === 'wifi') {
		const enc = wifi.value.encryption || 'WPA'
		const ssid = wifi.value.ssid || ''
		const pwd = wifi.value.password || ''
		if (!ssid) return ''
		if (enc === 'nopass') {
			return `WIFI:T:nopass;S:${ssid};;`
		}
		return `WIFI:T:${enc};S:${ssid};P:${pwd};;`
	}

	if (qrType.value === 'url') {
		let v = (textInput.value || '').trim()
		if (v && !/^https?:\/\//i.test(v)) v = 'https://' + v
		return v
	}

	// default: text
	return textInput.value || ''
}

const loading = ref(false)

// debounce settings: wait for user to stop typing before auto-generate
let debounceTimer = null
const debounceDelay = 800 // ms

// make generation intentionally feel a bit slower for UX (min spinner time)
const minSpinner = 700 // ms

const generate = async () => {
	if (!canvas.value) return
	const start = Date.now()
	loading.value = true
	try {
		const payload = buildPayload() || ' '
		// use fixed colors (dark on light)
		const color = { dark: '#000000', light: '#ffffff' }
		await QRCode.toCanvas(canvas.value, payload, { width: size.value, color })
		error.value = null
	} catch (err) {
		error.value = String(err.message || err)
	} finally {
		const elapsed = Date.now() - start
		const remaining = Math.max(0, minSpinner - elapsed)
		setTimeout(() => { loading.value = false }, remaining)
	}
}

function scheduleGenerate() {
	if (debounceTimer) clearTimeout(debounceTimer)
	debounceTimer = setTimeout(() => {
		debounceTimer = null
		generate()
	}, debounceDelay)
}

const download = () => {
	if (!canvas.value) return
	const link = document.createElement('a')
	link.href = canvas.value.toDataURL('image/png')
	link.download = 'qr.png'
	link.click()
}

// removed copyToClipboard — feature removed per user request

const clear = () => {
	textInput.value = ''
	wifi.value = { ssid: '', password: '', encryption: 'WPA' }
	if (canvas.value) {
		const ctx = canvas.value.getContext('2d')
		ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
	}
}

onMounted(() => {
	generate()
})

watch([qrType, textInput, () => wifi.value.ssid, () => wifi.value.password, () => wifi.value.encryption], () => {
	// don't auto-generate on every keystroke — wait until user idle
	scheduleGenerate()
})
</script>