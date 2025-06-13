<template>
    <div class="drawer lg:drawer-open">
        <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col items-center justify-center">
            <!-- Page content here -->
        </div>
        <div class="drawer-side">
            <h1 class="text-2xl font-bold text-center text-gray-300 mb-4 mx-auto mt-4">Shopping List</h1>
            <div class="w-full">
                <button class="btn mb-4 mx-auto block border-2 border-gray-300 rounded-md" @click="createNewList">Create
                    new list</button>
                <button @click="handleSignOut">Sign Out</button>
            </div>

            <label for="my-drawer-2" aria-label="close sidebar" class="drawer-overlay"></label>

            <h1 class="text-1xl ml-3 mb-2">Today & Yesterday</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li><a>Sidebar Item 1</a></li>
                <li><a>Sidebar Item 2</a></li>
            </ul>

            <h1 class="text-1xl ml-3 mb-2">Older than 3 days</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li><a>Older Item 1</a></li>
            </ul>

            <h1 class="text-1xl ml-3 mb-2">Favorites</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li><a>Favorite Item 1</a></li>
            </ul>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useDebounceFn } from '@vueuse/core'

const { logout } = useSanctumAuth()
const shoppingLists = ref([])


// Helper function to get CSRF token from cookie
function getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
}

async function createNewList() {
    try {
        // Get CSRF cookie
        await $fetch('http://localhost:8000/sanctum/csrf-cookie', {
            credentials: 'include'
        });

        // Get the CSRF token from the cookie
        const xsrfToken = decodeURIComponent(getCookie('XSRF-TOKEN'));

        // Create the list
        const newList = await $fetch('http://localhost:8000/api/shopping-lists', {
            method: 'POST',
            headers: {
                'Accept': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            body: {
                name: 'New Shopping List'
            },
            credentials: 'include'
        });

        alert('Shopping list created!');
    } catch (error) {
        console.error('Error creating new list:', error);
        alert('Error creating new list. Please try again.');
    }
}

async function handleSignOut() {
    try {
        await logout()
        navigateTo('/auth/login')
    } catch (error) {
        console.error('Error signing out:', error)
        alert('Error signing out. Please try again.')
    }
}

onMounted(async () => {
    try {
        const response = await $fetch('http://localhost:8000/api/shopping-lists', {
            credentials: 'include'
        })
        shoppingLists.value = response
        console.log(response)
    } catch (error) {
        console.error('Error fetching shopping lists:', error)
        alert('Error loading shopping lists. Please try again.')
    }
})


</script>