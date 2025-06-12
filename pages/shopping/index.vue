<template>
    <div>
      <pre>{{ data }}</pre>
    </div>
  </template>

<script setup>
    const {data } = await useFetch('/api/shopping-lists', {
        baseURL: 'https://localhost:8000',
        credentials: 'include'
    })

</script>