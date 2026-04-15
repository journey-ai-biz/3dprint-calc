<script setup>
import { ref, computed } from 'vue'

const defaults = {
  filamentPrice: 1000,
  weight: 120,
  printTime: 4.5,
  powerDraw: 80,
  electricityRate: 8,
  maintenanceRate: 15,
  setupFee: 50,
  failBuffer: 5,
  markup: 25,
}

const filamentPrice = ref(defaults.filamentPrice)
const weight = ref(defaults.weight)
const printTime = ref(defaults.printTime)
const powerDraw = ref(defaults.powerDraw)
const electricityRate = ref(defaults.electricityRate)
const maintenanceRate = ref(defaults.maintenanceRate)
const setupFee = ref(defaults.setupFee)
const failBuffer = ref(defaults.failBuffer)
const markup = ref(defaults.markup)

const copied = ref(false)

const calculations = computed(() => {
  const matBase = (weight.value / 1000) * filamentPrice.value
  const matWithBuffer = matBase * (1 + failBuffer.value / 100)
  const elec = (powerDraw.value / 1000) * printTime.value * electricityRate.value
  const wear = printTime.value * maintenanceRate.value

  const totalCost = matWithBuffer + elec + wear + setupFee.value
  const finalPrice = totalCost * (1 + markup.value / 100)

  return { matWithBuffer, elec, wear, totalCost, finalPrice }
})

function copyQuote() {
  const text = `₹${Math.round(calculations.value.finalPrice)}`
  if (navigator.clipboard) {
    navigator.clipboard.writeText(text).then(() => {
      copied.value = true
      setTimeout(() => (copied.value = false), 1800)
    })
  }
}

function resetInputs() {
  filamentPrice.value = defaults.filamentPrice
  weight.value = defaults.weight
  printTime.value = defaults.printTime
  powerDraw.value = defaults.powerDraw
  electricityRate.value = defaults.electricityRate
  maintenanceRate.value = defaults.maintenanceRate
  setupFee.value = defaults.setupFee
  failBuffer.value = defaults.failBuffer
  markup.value = defaults.markup
}
</script>

<template>
  <div class="min-h-screen p-6 bg-gradient-to-b from-slate-50 to-white flex items-center">
    <div class="max-w-4xl w-full mx-auto bg-white rounded-2xl shadow-2xl overflow-hidden grid grid-cols-1 md:grid-cols-3">
      <div class="md:col-span-2 p-8">
        <header class="flex items-start justify-between gap-4">
          <div>
            <h1 class="text-2xl font-extrabold text-slate-900">3D Print Quote</h1>
            <p class="text-sm text-slate-500 mt-1">Quick quotes for Bambu Lab P1S — tweak values to refine pricing.</p>
          </div>
          <div class="text-right space-y-2">
            <button @click="copyQuote" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-md">Copy Quote</button>
            <button @click="resetInputs" class="border border-slate-200 px-3 py-2 rounded-md text-sm">Reset</button>
            <div v-if="copied" class="copy-badge">Copied</div>
          </div>
        </header>

        <div class="mt-6 grid grid-cols-1 lg:grid-cols-2 gap-6">
          <section class="space-y-4">
            <h2 class="text-sm font-semibold text-slate-600 uppercase">Job Details</h2>
            <div class="grid grid-cols-1 gap-4">
              <label class="text-xs text-slate-600">Filament Price (₹/kg)
                <input v-model="filamentPrice" type="number" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>

              <label class="text-xs text-slate-600">Weight (Grams)
                <input v-model="weight" type="number" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>

              <label class="text-xs text-slate-600">Print Time (Hours)
                <input v-model="printTime" type="number" step="0.5" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>
            </div>

            <h2 class="text-sm font-semibold text-slate-600 uppercase mt-4">Machine & Overheads</h2>
            <div class="grid grid-cols-1 gap-4">
              <label class="text-xs text-slate-600">Power Draw (W)
                <input v-model="powerDraw" type="number" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>
              <label class="text-xs text-slate-600">Electricity Rate (₹/kWh)
                <input v-model="electricityRate" type="number" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>
              <label class="text-xs text-slate-600">Maintenance Rate (₹/hour)
                <input v-model="maintenanceRate" type="number" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>
              <label class="text-xs text-slate-600">Setup Fee (₹)
                <input v-model="setupFee" type="number" class="mt-1 w-full rounded-md border border-slate-200 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400" />
              </label>
            </div>

            <div class="flex flex-col gap-3 mt-4">
              <label class="flex justify-between text-xs text-slate-600">
                <span>Fail Buffer</span>
                <span>{{ failBuffer }}%</span>
              </label>
              <input v-model="failBuffer" type="range" min="0" max="30" class="w-full h-2" />

              <label class="flex justify-between text-xs text-slate-600 mt-2">
                <span>Profit Markup</span>
                <span>{{ markup }}%</span>
              </label>
              <input v-model="markup" type="range" min="0" max="200" class="w-full h-2" />
            </div>
          </section>

          <aside class="bg-gradient-to-b from-slate-900 to-slate-800 text-white rounded-xl p-6 flex flex-col justify-between">
            <div>
              <h3 class="text-sm uppercase text-slate-300 tracking-wider text-center">Price Breakdown</h3>

              <div class="mt-4 space-y-3 text-sm">
                <div class="flex justify-between">
                  <span class="text-slate-400">Material (Inc. Buffer)</span>
                  <span>₹{{ calculations.matWithBuffer.toFixed(2) }}</span>
                </div>
                <div class="flex justify-between">
                  <span class="text-slate-400">Electricity</span>
                  <span>₹{{ calculations.elec.toFixed(2) }}</span>
                </div>
                <div class="flex justify-between border-b border-slate-700 pb-3">
                  <span class="text-slate-400">Machine Wear</span>
                  <span>₹{{ calculations.wear.toFixed(2) }}</span>
                </div>
                <div class="flex justify-between pt-2">
                  <span class="text-slate-400">Base Cost</span>
                  <span class="font-mono">₹{{ calculations.totalCost.toFixed(2) }}</span>
                </div>
              </div>
            </div>

            <div class="mt-6 text-center">
              <div class="text-xs text-slate-400 uppercase font-semibold">Total Quote</div>
              <div class="text-4xl font-extrabold text-green-300 mt-3">₹{{ Math.round(calculations.finalPrice) }}</div>
            </div>
          </aside>
        </div>
      </div>
    </div>
  </div>
</template>
