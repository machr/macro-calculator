<template>
  <form @submit="handleSubmit">
    <input name="name" placeholder="Name" :value="values.name" @input="handleFieldChange" />
    <button type="submit">Submit</button>
  </form>
</template>
<script setup lang="ts">
import { useFormik } from 'vue-formik'
const { values, handleFieldChange, handleSubmit } = useFormik({
  initialValues: {
    name: '',
  },
  onSubmit: (values) => {
    alert(JSON.stringify(values, null, 2))
    // { "name": "John Doe" }
  },
})
</script>
