<script setup>
import { ref, computed } from 'vue'

// --- State ---
const filamentPrice = ref(1500)
const weight = ref(120)
const printTime = ref(4.5)
const powerDraw = ref(80)
const electricityRate = ref(8)
const maintenanceRate = ref(15)
const setupFee = ref(50)
const failBuffer = ref(5)
const markup = ref(25)

// --- Logic ---
const calculations = computed(() => {
  const matBase = (weight.value / 1000) * filamentPrice.value
  const matWithBuffer = matBase * (1 + failBuffer.value / 100)
  const elec = (powerDraw.value / 1000) * printTime.value * electricityRate.value
  const wear = printTime.value * maintenanceRate.value
  
  const totalCost = matWithBuffer + elec + wear + setupFee.value
  const finalPrice = totalCost * (1 + markup.value / 100)
  
  return { matWithBuffer, elec, wear, totalCost, finalPrice }
})
</script>

<template>
  <div class="min-h-screen bg-slate-50 p-4 md:p-8 font-sans text-slate-900">
    <div class="max-w-2xl mx-auto bg-white rounded-2xl shadow-xl overflow-hidden">
      <div class="bg-blue-600 p-6 text-white">
        <h1 class="text-2xl font-bold">3D Print Quote Generator</h1>
        <p class="opacity-80 text-sm">Fine-tuned for Bambu Lab P1S</p>
      </div>

      <div class="p-6 grid grid-cols-1 md:grid-cols-2 gap-8">
        <div class="space-y-6">
          <section>
            <h2 class="text-sm font-bold uppercase tracking-wider text-slate-400 mb-4">Job Details</h2>
            <div class="space-y-3">
              <div>
                <label class="text-xs font-semibold">Filament Price (₹/kg)</label>
                <input v-model="filamentPrice" type="number" class="w-full border-b-2 border-slate-200 focus:border-blue-500 outline-none py-1 transition-colors" />
              </div>
              <div>
                <label class="text-xs font-semibold">Weight (Grams)</label>
                <input v-model="weight" type="number" class="w-full border-b-2 border-slate-200 focus:border-blue-500 outline-none py-1 transition-colors" />
              </div>
              <div>
                <label class="text-xs font-semibold">Print Time (Hours)</label>
                <input v-model="printTime" type="number" step="0.5" class="w-full border-b-2 border-slate-200 focus:border-blue-500 outline-none py-1 transition-colors" />
              </div>
            </div>
          </section>

          <section>
            <h2 class="text-sm font-bold uppercase tracking-wider text-slate-400 mb-4">Overheads</h2>
            <div class="space-y-4">
              <div>
                <label class="flex justify-between text-xs font-semibold">
                  <span>Fail Buffer</span>
                  <span>{{ failBuffer }}%</span>
                </label>
                <input v-model="failBuffer" type="range" min="0" max="30" class="w-full h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-blue-600" />
              </div>
              <div>
                <label class="flex justify-between text-xs font-semibold">
                  <span>Profit Markup</span>
                  <span>{{ markup }}%</span>
                </label>
                <input v-model="markup" type="range" min="0" max="200" class="w-full h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-green-600" />
              </div>
            </div>
          </section>
        </div>

        <div class="bg-slate-900 rounded-xl p-6 text-white flex flex-col justify-between shadow-inner">
          <div>
            <h2 class="text-slate-400 text-sm font-bold uppercase mb-6 text-center">Price Breakdown</h2>
            <div class="space-y-3 text-sm">
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

          <div class="mt-8 text-center border-t border-slate-700 pt-6">
            <span class="text-slate-400 block text-xs uppercase font-bold mb-1">Total Quote</span>
            <span class="text-5xl font-black text-green-400">₹{{ Math.round(calculations.finalPrice) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>