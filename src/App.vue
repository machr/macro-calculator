<script setup lang="ts">
import { computed, defineModel } from 'vue'

type Gender = 'male' | 'female'

// Input Models
const height = defineModel<number>('height', { required: true, default: 180 })
const weight = defineModel<number>('weight', { required: true, default: 75 })
const age = defineModel<number>('age', { required: true, default: 35 })
const gender = defineModel<Gender>('gender', { required: true, default: 'female' })

// Calculation Computed
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
</script>

<template>
  <div class="max-w-md mx-auto p-4 space-y-4">
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
        <select v-model="gender" class="border border-zinc-500 ml-auto">
          <option value="male">Male</option>
          <option value="female">Female</option>
        </select>
      </label>
    </div>

    <div class="p-4 border rounded">
      <p>
        Mifflin-St Jeor: <strong>{{ rmr.mifflin }}</strong> kcal
      </p>
      <p>
        Harris-Benedict: <strong>{{ rmr.harris }}</strong> kcal
      </p>
      <p>
        Average Estimate: <strong>{{ rmr.average }}</strong> kcal
      </p>
    </div>
  </div>
</template>
