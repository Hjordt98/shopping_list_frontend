<template>
    <div class="drawer lg:drawer-open">
        <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col items-center justify-center">
            <div class="flex flex-col items-center justify-center w-full mt-10 px-4">
                <textarea placeholder="Type here"
                    class="bg-base-200 border border-gray-400 rounded-lg p-4 text-lg w-full max-w-6xl h-[675px] resize-none focus:outline-none focus:ring-2 focus:ring-primary"
                    style="box-sizing: border-box; overflow-x: hidden; overflow-y: auto;"></textarea>
            </div>
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
                <li v-for="list in todayAndYesterday" :key="list.id">
                    <a> {{ list.name }} </a>
                </li>
            </ul>

            <h1 class="text-1xl ml-3 mb-2">Older than 3 days</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in olderList" :key="list.id">
                    <a>{{ list.name }}</a>
                </li>
            </ul>

            <h1 class="text-1xl ml-3 mb-2">Favorites</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li><a>Favorite Item 1</a></li>
            </ul>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useDebounceFn } from '@vueuse/core'

const { logout } = useSanctumAuth()

// ref to handle computed properties
const shoppingLists = ref([])

// computed properties for today and yesterday
const todayAndYesterday = computed(() => {
    const now = new Date();
    // Get UTC timestamp for now
    const nowUTC = now.getTime() - now.getTimezoneOffset() * 60000;
    // 48 hours ago in UTC
    const twoDaysAgoUTC = nowUTC - 48 * 60 * 60 * 1000;

    return shoppingLists.value.filter(list => {
        const listUpdatedDate = new Date(list.updated_at);
        const listUTC = listUpdatedDate.getTime();
        return listUTC >= twoDaysAgoUTC && listUTC <= nowUTC;
    }).sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
})

const olderList = computed(() => {
    const threeDaysAgo = new Date()
    threeDaysAgo.setDate(threeDaysAgo.getDate() - 3)

    return shoppingLists.value
        .filter(list => {
            const listUpdatedDate = new Date(list.updated_at)
            return listUpdatedDate < threeDaysAgo
        })
        .sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
})


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