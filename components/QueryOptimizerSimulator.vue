<script setup>
import { ref, computed } from 'vue'

const totalRows = 1000000
const selectivity = ref(5) // Percentage of rows matching 'age > 25'

// Hypothetical cost constants
const seqIoCost = 0.1      // Cost of sequential read per block/row
const randomIoCost = 4.0   // Cost of random read per row
const indexTraverse = 50.0 // Base cost to traverse B-tree to first leaf

const matchingRows = computed(() => Math.floor(totalRows * (selectivity.value / 100)))

// Full Table Scan Cost: Read all rows sequentially
const tableScanCost = computed(() => {
  return totalRows * seqIoCost
})

// Index Seek Cost: Traverse index + random read for each matching row
const indexSeekCost = computed(() => {
  return indexTraverse + (matchingRows.value * randomIoCost)
})

const chosenPlan = computed(() => {
  return indexSeekCost.value < tableScanCost.value ? 'Index Seek' : 'Full Table Scan'
})
</script>

<template>
  <div class="p-6 border border-gray-300 dark:border-gray-700 rounded-lg shadow-lg bg-white dark:bg-gray-900 mt-4 text-sm">
    <h3 class="text-xl font-bold mb-4 text-center">Query Optimizer Cost Simulator</h3>
    
    <div class="mb-6">
      <label class="block mb-2 font-semibold">Selectivity of <code>age > 25</code>: <span class="text-blue-500">{{ selectivity }}%</span> ({{ matchingRows.toLocaleString() }} rows)</label>
      <input 
        type="range" 
        v-model="selectivity" 
        min="0" max="100" 
        class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
      >
      <div class="flex justify-between text-xs opacity-70 mt-1">
        <span>0%</span>
        <span>50%</span>
        <span>100%</span>
      </div>
    </div>

    <div class="grid grid-cols-2 gap-4 mb-6">
      <div 
        class="p-4 rounded-md border transition-all duration-300"
        :class="chosenPlan === 'Full Table Scan' ? 'bg-green-100 dark:bg-green-900/30 border-green-500' : 'bg-gray-50 dark:bg-gray-800 border-gray-200 dark:border-gray-700 opacity-50'"
      >
        <h4 class="font-bold mb-2">Full Table Scan</h4>
        <div class="text-xs opacity-80 mb-2">Cost Model: Sequential Read (1M rows)</div>
        <div class="text-lg font-mono">{{ tableScanCost.toLocaleString() }}</div>
      </div>
      
      <div 
        class="p-4 rounded-md border transition-all duration-300"
        :class="chosenPlan === 'Index Seek' ? 'bg-green-100 dark:bg-green-900/30 border-green-500' : 'bg-gray-50 dark:bg-gray-800 border-gray-200 dark:border-gray-700 opacity-50'"
      >
        <h4 class="font-bold mb-2">Index Seek</h4>
        <div class="text-xs opacity-80 mb-2">Cost Model: B-Tree + Random Read ({{ matchingRows.toLocaleString() }} rows)</div>
        <div class="text-lg font-mono">{{ indexSeekCost.toLocaleString() }}</div>
      </div>
    </div>

    <div class="text-center p-3 rounded bg-blue-50 dark:bg-blue-900/20 text-blue-800 dark:text-blue-200 font-semibold text-lg">
      Optimizer Choice: <span class="font-bold underline">{{ chosenPlan }}</span>
    </div>
    
    <div class="text-center text-xs opacity-60 mt-3">
      *Notice how at around ~2.5% selectivity, the cost of many random reads exceeds a sequential full table scan!
    </div>
  </div>
</template>
