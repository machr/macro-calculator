<script setup lang="ts">
import { ref, computed } from 'vue'

type Gender = 'male' | 'female'

// User Inputs
const height = ref<number>(182) // cm
const weight = ref<number>(85.3) // kg
const age = ref<number>(39)
const gender = ref<Gender>('female')
const activityLevel = ref<string>('sedentary') // Default activity level

// TDEE Multiplier by Activity Level
const activityMultipliers: { [key: string]: number } = {
  'bed-rest': 1.2,
  'very-sedentary': 1.3,
  sedentary: 1.4,
  light: 1.5,
  'light-moderate': gender.value === 'male' ? 1.7 : 1.6,
  moderate: gender.value === 'male' ? 1.8 : 1.7,
  heavy: gender.value === 'male' ? 2.1 : 1.8,
  'very-heavy': gender.value === 'male' ? 2.3 : 2.0,
}

// RMR Calculation
const rmr = computed(() => {
  const mifflin =
    10 * weight.value + 6.25 * height.value - 5 * age.value + (gender.value === 'male' ? 5 : -161)
  const harris =
    gender.value === 'male'
      ? 88.362 + 13.397 * weight.value + 4.799 * height.value - 5.677 * age.value
      : 447.593 + 9.247 * weight.value + 3.098 * height.value - 4.33 * age.value

  const average = (mifflin + harris) / 2
  return {
    mifflin: Math.round(mifflin),
    harris: Math.round(harris),
    average: Math.round(average),
  }
})

// TDEE Calculation (using activity multiplier)
const tdee = computed(() => {
  const bmr = rmr.value.average
  const multiplier = activityMultipliers[activityLevel.value]
  return Math.round(bmr * multiplier)
})

// Body Composition TDEE
const bodyCompositionTdee = computed(() => ({
  weightLoss: tdee.value - 500,
  maintenance: tdee.value,
  weightGain: tdee.value + 500,
}))

// Macronutrient calculations
const proteinPerKg = ref<number>(1.8)
const fatPerKg = ref<number>(1.0)
const carbsPerKg = ref<number>(3.5)

const macros = computed(() => {
  const proteinGrams = Math.round(weight.value * proteinPerKg.value)
  const fatGrams = Math.round(weight.value * fatPerKg.value)
  const carbsGrams = Math.round(weight.value * carbsPerKg.value)

  const proteinCalories = proteinGrams * 4
  const fatCalories = fatGrams * 9
  const carbsCalories = carbsGrams * 4

  const totalCalories = proteinCalories + fatCalories + carbsCalories

  return {
    protein: {
      grams: proteinGrams,
      calories: proteinCalories,
      percentage: Math.round((proteinCalories / totalCalories) * 100),
    },
    fat: {
      grams: fatGrams,
      calories: fatCalories,
      percentage: Math.round((fatCalories / totalCalories) * 100),
    },
    carbs: {
      grams: carbsGrams,
      calories: carbsCalories,
      percentage: Math.round((carbsCalories / totalCalories) * 100),
    },
    totalCalories,
  }
})
</script>

<template>
  <div class="w-full min-h-screen flex items-center justify-center bg-gray-50">
    <div class="w-full max-w-[1536px] px-4 sm:px-6 md:px-8 lg:px-10 xl:px-12 2xl:px-16">
      <div class="flex justify-center">
        <div
          class="w-full max-w-md sm:max-w-lg md:max-w-xl lg:max-w-2xl xl:max-w-3xl 2xl:max-w-4xl p-6 flex flex-col gap-8 bg-white shadow-lg rounded-xl"
        >
          <h2 class="text-2xl font-bold text-gray-800">Macro Calculator</h2>

          <div class="flex flex-col gap-6 bg-gray-50 p-6 rounded-lg">
            <h3 class="text-lg font-semibold text-gray-700">RMR Calculation</h3>
            <div class="space-y-4">
              <label class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                <span class="text-gray-700">Height (cm)</span>
                <input
                  v-model.number="height"
                  type="number"
                  class="w-full sm:w-32 px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                />
              </label>

              <label class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                <span class="text-gray-700">Weight (kg)</span>
                <input
                  v-model.number="weight"
                  type="number"
                  min="30"
                  max="350"
                  class="w-full sm:w-32 px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                />
              </label>

              <label class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                <span class="text-gray-700">Age</span>
                <input
                  v-model.number="age"
                  type="number"
                  class="w-full sm:w-32 px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                />
              </label>

              <label class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                <span class="text-gray-700">Gender</span>
                <select
                  v-model="gender"
                  class="w-full sm:w-32 px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >
                  <option value="male">Male</option>
                  <option value="female">Female</option>
                </select>
              </label>
            </div>

            <div class="mt-4">
              <h3 class="text-lg font-semibold text-gray-700 mb-3">Activity Level</h3>
              <label class="block">
                <select
                  v-model="activityLevel"
                  class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >
                  <option value="bed-rest">Bed Rest</option>
                  <option value="very-sedentary">Very Sedentary</option>
                  <option value="sedentary">Sedentary/Maintenance</option>
                  <option value="light">Light</option>
                  <option value="light-moderate">Light/Moderate</option>
                  <option value="moderate">Moderate</option>
                  <option value="heavy">Heavy</option>
                  <option value="very-heavy">Very Heavy</option>
                </select>
              </label>
            </div>
          </div>

          <!-- Results Section -->
          <div class="overflow-x-auto bg-gray-50 p-6 rounded-lg">
            <h3 class="text-lg font-semibold text-gray-700 mb-4">RMR Results</h3>
            <table class="min-w-full table-auto">
              <thead>
                <tr class="border-b border-gray-200">
                  <th class="py-3 px-4 text-left font-semibold text-gray-700">Calculation</th>
                  <th class="py-3 px-4 text-right font-semibold text-gray-700">Value (kcal)</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200">
                <tr>
                  <td class="py-3 px-4 text-gray-600">Mifflin-St Jeor</td>
                  <td class="py-3 px-4 text-right">
                    <strong class="text-gray-800">{{ rmr.mifflin }}</strong>
                  </td>
                </tr>
                <tr>
                  <td class="py-3 px-4 text-gray-600">Harris-Benedict</td>
                  <td class="py-3 px-4 text-right">
                    <strong class="text-gray-800">{{ rmr.harris }}</strong>
                  </td>
                </tr>
                <tr>
                  <td class="py-3 px-4 text-gray-600">Average Estimate</td>
                  <td class="py-3 px-4 text-right">
                    <strong class="text-gray-800">{{ rmr.average }}</strong>
                  </td>
                </tr>
                <tr class="border-t border-gray-200">
                  <td class="py-3 px-4 font-semibold text-gray-700">TDEE (with activity)</td>
                  <td class="py-3 px-4 text-right">
                    <span class="font-bold text-blue-600">{{ tdee }}</span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Body Composition Section -->
          <div class="overflow-x-auto bg-gray-50 p-6 rounded-lg">
            <h3 class="text-lg font-semibold text-gray-700 mb-4">Body Composition Approach</h3>
            <table class="min-w-full table-auto">
              <thead>
                <tr class="border-b border-gray-200">
                  <th class="py-3 px-4 text-left font-semibold text-gray-700">Goal</th>
                  <th class="py-3 px-4 text-right font-semibold text-gray-700">Daily Calories</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200">
                <tr>
                  <td class="py-3 px-4 text-gray-600">Weight Loss (-500 kcal)</td>
                  <td class="py-3 px-4 text-right">
                    <strong class="text-red-600">{{ bodyCompositionTdee.weightLoss }}</strong>
                  </td>
                </tr>
                <tr>
                  <td class="py-3 px-4 text-gray-600">Maintenance</td>
                  <td class="py-3 px-4 text-right">
                    <strong class="text-gray-800">{{ bodyCompositionTdee.maintenance }}</strong>
                  </td>
                </tr>
                <tr>
                  <td class="py-3 px-4 text-gray-600">Weight Gain (+500 kcal)</td>
                  <td class="py-3 px-4 text-right">
                    <strong class="text-green-600">{{ bodyCompositionTdee.weightGain }}</strong>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Macronutrient Calculator Section -->
          <div class="overflow-x-auto bg-gray-50 p-6 rounded-lg">
            <h3 class="text-lg font-semibold text-gray-700 mb-4">Macronutrient Calculator</h3>

            <div class="space-y-6">
              <!-- Protein -->
              <div>
                <div class="flex justify-between items-center mb-2">
                  <label class="text-gray-700">Protein (g/kg)</label>
                  <span class="text-sm text-gray-500">Recommended: 1.6-2.2g/kg</span>
                </div>
                <input
                  v-model.number="proteinPerKg"
                  type="range"
                  min="1.6"
                  max="2.2"
                  step="0.1"
                  class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
                />
                <div class="flex justify-between text-sm text-gray-500 mt-1">
                  <span>1.6g</span>
                  <span>{{ proteinPerKg.toFixed(1) }}g</span>
                  <span>2.2g</span>
                </div>
              </div>

              <!-- Fat -->
              <div>
                <div class="flex justify-between items-center mb-2">
                  <label class="text-gray-700">Fat (g/kg)</label>
                  <span class="text-sm text-gray-500">Recommended: 0.8-1.2g/kg</span>
                </div>
                <input
                  v-model.number="fatPerKg"
                  type="range"
                  min="0.8"
                  max="1.2"
                  step="0.1"
                  class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
                />
                <div class="flex justify-between text-sm text-gray-500 mt-1">
                  <span>0.8g</span>
                  <span>{{ fatPerKg.toFixed(1) }}g</span>
                  <span>1.2g</span>
                </div>
              </div>

              <!-- Carbs -->
              <div>
                <div class="flex justify-between items-center mb-2">
                  <label class="text-gray-700">Carbohydrates (g/kg)</label>
                  <span class="text-sm text-gray-500">Recommended: 3-5g/kg</span>
                </div>
                <input
                  v-model.number="carbsPerKg"
                  type="range"
                  min="3"
                  max="5"
                  step="0.1"
                  class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
                />
                <div class="flex justify-between text-sm text-gray-500 mt-1">
                  <span>3g</span>
                  <span>{{ carbsPerKg.toFixed(1) }}g</span>
                  <span>5g</span>
                </div>
              </div>
            </div>

            <!-- Results Table -->
            <div class="mt-6">
              <table class="min-w-full table-auto">
                <thead>
                  <tr class="border-b border-gray-200">
                    <th class="py-3 px-4 text-left font-semibold text-gray-700">Macro</th>
                    <th class="py-3 px-4 text-right font-semibold text-gray-700">Grams</th>
                    <th class="py-3 px-4 text-right font-semibold text-gray-700">Calories</th>
                    <th class="py-3 px-4 text-right font-semibold text-gray-700">%</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-gray-200">
                  <tr>
                    <td class="py-3 px-4 text-gray-600">Protein</td>
                    <td class="py-3 px-4 text-right">{{ macros.protein.grams }}g</td>
                    <td class="py-3 px-4 text-right">{{ macros.protein.calories }} kcal</td>
                    <td class="py-3 px-4 text-right">{{ macros.protein.percentage }}%</td>
                  </tr>
                  <tr>
                    <td class="py-3 px-4 text-gray-600">Fat</td>
                    <td class="py-3 px-4 text-right">{{ macros.fat.grams }}g</td>
                    <td class="py-3 px-4 text-right">{{ macros.fat.calories }} kcal</td>
                    <td class="py-3 px-4 text-right">{{ macros.fat.percentage }}%</td>
                  </tr>
                  <tr>
                    <td class="py-3 px-4 text-gray-600">Carbohydrates</td>
                    <td class="py-3 px-4 text-right">{{ macros.carbs.grams }}g</td>
                    <td class="py-3 px-4 text-right">{{ macros.carbs.calories }} kcal</td>
                    <td class="py-3 px-4 text-right">{{ macros.carbs.percentage }}%</td>
                  </tr>
                  <tr class="border-t border-gray-200">
                    <td class="py-3 px-4 font-semibold text-gray-700">Total</td>
                    <td class="py-3 px-4 text-right">
                      {{ macros.protein.grams + macros.fat.grams + macros.carbs.grams }}g
                    </td>
                    <td class="py-3 px-4 text-right font-semibold">
                      {{ macros.totalCalories }} kcal
                    </td>
                    <td class="py-3 px-4 text-right">100%</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
