<template>
    <div class="min-h-screen flex items-center justify-center">
      <div class="card w-96 bg-base-100 shadow-xl">
        <div class="card-body">
          <h2 class="card-title">Register</h2>
          <form @submit.prevent="handleRegister">
            <div class="form-control">
              <label class="label">
                <span class="label-text">Name</span>
              </label>
              <input 
                type="text" 
                v-model="name" 
                class="input input-bordered" 
                required
              />
            </div>
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
            <div class="form-control">
              <label class="label">
                <span class="label-text">Confirm Password</span>
              </label>
              <input 
                type="password" 
                v-model="confirmPassword" 
                class="input input-bordered" 
                required
              />
            </div>
            <div class="form-control mt-6">
              <button type="submit" class="btn btn-primary">Register</button>
              <p class="text-gray-400 text-sm mt-2">Already have an account? <NuxtLink to="/auth/login" class="text-blue-500">Login</NuxtLink></p>
            </div>
          </form>
        </div>
      </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const name = ref('')
const email = ref('')
const password = ref('')
const confirmPassword = ref('')

function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

const handleRegister = async () => {
  try {
    // 1. Get CSRF cookie
    await $fetch('http://localhost:8000/sanctum/csrf-cookie', { credentials: 'include' })

    // 2. Extract XSRF token from cookie
    const xsrfToken = getCookie('XSRF-TOKEN')
    console.log('Extracted XSRF-TOKEN:', xsrfToken)

    // 3. Register user with XSRF token header
    await $fetch('http://localhost:8000/register', {
      method: 'POST',
      body: {
        name: name.value,
        email: email.value,
        password: password.value,
        password_confirmation: confirmPassword.value
      },
      credentials: 'include',
      headers: {
        'X-XSRF-TOKEN': decodeURIComponent(xsrfToken)
      }
    })

    alert('Registration successful! You can now log in.')
    navigateTo('/auth/login')
  } catch (error) {
    console.error('Registration failed:', error)
    alert('Registration failed. See console for details.')
  }
}
</script>