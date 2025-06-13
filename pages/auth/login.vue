<template>
  <div class="min-h-screen flex items-center justify-center">
    <div class="card w-96 bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title">Login</h2>
        <form @submit.prevent="handleLogin">
          <div class="form-control">
            <label class="label">
              <span class="label-text">Email</span>
            </label>
            <input 
              type="email" 
              v-model="email" 
              class="input input-bordered" 
              required
            />
          </div>
          <div class="form-control">
            <label class="label">
              <span class="label-text">Password</span>
            </label>
            <input 
              type="password" 
              v-model="password" 
              class="input input-bordered" 
              required
            />
          </div>
          <div class="form-control mt-6">
            <button type="submit" class="btn btn-primary">Login</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
const { login } = useSanctumAuth()
const email = ref('')
const password = ref('')

const handleLogin = async () => {
  try {
    await $fetch('http://localhost:8000/sanctum/csrf-cookie', { credentials: 'include' })
    await login({ email: email.value, password: password.value })
    navigateTo('/')
  } catch (error) {
    alert('Login failed. Please check your credentials.')
    console.error('Login error:', error)
  }
}
</script> 