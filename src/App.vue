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
</script>

<template>
  <div class="max-w-md mx-auto p-4 flex flex-col gap-10">
    <h2 class="text-xl font-bold">Macro Calculator</h2>

    <div class="flex flex-col gap-4">
      <label class="flex justify-between">
        <span>Height (cm)</span>
        <input v-model.number="height" type="number" class="border border-zinc-500 ml-auto" />
      </label>

      <label class="flex justify-between">
        <span>Weight (kg)</span>
        <input
          v-model.number="weight"
          type="number"
          min="30"
          max="350"
          class="border border-zinc-500 ml-auto"
        />
      </label>

      <label class="flex justify-between">
        <span>Age</span>
        <input v-model.number="age" type="number" class="border border-zinc-500 ml-auto" />
      </label>

      <label class="flex justify-between">
        Gender
        <select v-model="gender" class="border border-zinc-500 ml-auto px-2 py-1">
          <option value="male">Male</option>
          <option value="female">Female</option>
        </select>
      </label>

      <label class="flex justify-between">
        Activity Level
        <select v-model="activityLevel" class="border border-zinc-500 ml-auto px-2 py-1">
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

    <div class="overflow-x-auto p-4 border rounded">
      <table class="min-w-full table-auto">
        <thead>
          <tr>
            <th class="py-2 px-4 text-left font-semibold">Calculation</th>
            <th class="py-2 px-4 text-right font-semibold">Value (kcal)</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="py-2 px-4">Mifflin-St Jeor</td>
            <td class="py-2 px-4 text-right">
              <strong>{{ rmr.mifflin }}</strong>
            </td>
          </tr>
          <tr>
            <td class="py-2 px-4">Harris-Benedict</td>
            <td class="py-2 px-4 text-right">
              <strong>{{ rmr.harris }}</strong>
            </td>
          </tr>
          <tr>
            <td class="py-2 px-4">Average Estimate</td>
            <td class="py-2 px-4 text-right">
              <strong>{{ rmr.average }}</strong>
            </td>
          </tr>
          <tr class="border-t">
            <td class="py-2 px-4 font-semibold">TDEE (with activity)</td>
            <td class="py-2 px-4 text-right font-semibold">
              <span class="font-bold">{{ tdee }}</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
