<template>
    <div class="drawer lg:drawer-open">
        <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col items-center justify-center">
            <div class="flex flex-col items-center justify-center w-full mt-10 px-4">
                <textarea class="
                        textarea textarea-ghost    <!-- Base textarea styles -->
                        bg-base-200               <!-- Background color -->
                        border                    <!-- Border -->
                        border-gray-400           <!-- Border color -->
                        rounded-lg                <!-- Rounded corners -->
                        p-4                       <!-- Padding -->
                        text-lg                   <!-- Text size -->
                        resize-none               <!-- Prevent resizing -->
                        focus:outline-none        <!-- Remove focus outline -->
                        focus:ring-2              <!-- Focus ring width -->
                        max-w-6xl
                        w-full
                        focus:ring-primary        <!-- Focus ring color -->
                        mb-8                      <!-- Bottom margin -->
                        text-center
                    " placeholder="Name of shopping list" v-model="selectedListName"></textarea>
                <textarea placeholder="Type here" class="
                        bg-base-200                <!-- Background color (from your theme) -->
                        border                     <!-- Adds a border -->
                        border-gray-400            <!-- Border color -->
                        rounded-lg                 <!-- Large rounded corners -->
                        p-4                        <!-- Padding on all sides -->
                        text-lg                    <!-- Large text size -->
                        w-full                     <!-- Full width of parent -->
                        max-w-6xl                  <!-- Maximum width (very wide, but responsive) -->
                        h-[675px]                  <!-- Fixed height: 675px -->
                        resize-none                <!-- Prevents user from resizing the textarea -->
                        focus:outline-none         <!-- Removes default focus outline -->
                    " style="
                        box-sizing: border-box;    /* Ensures padding/border are included in width/height */
                        overflow-x: hidden;        /* Prevents horizontal scroll */
                        overflow-y: auto;          /* Allows vertical scroll if content overflows */
                    " v-model="textareaValue" "></textarea>
                       <button @click="deleteList(selectedListId)" class="btn btn-ghost btn absolute bottom-0 right-0 mb-10 mr-10">delete</button>
            </div>
         
        </div>
        <div class=" drawer-side">
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
                    <a @click="handleListClick(list.id)"> {{ truncateListName(list.name) }} </a>
                    </li>
                </ul>
   

            <h1 class="text-1xl ml-3 mb-2">Older than 3 days</h1>
        
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in olderList" :key="list.id">
                    <a @click="handleListClick(list.id)">{{ truncateListName(list.name) }}</a>
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

// imports
import { ref, onMounted, computed, watch } from 'vue'
import { useDebounceFn } from '@vueuse/core'

// useSanctumAuth hook to handle authentication
const { logout } = useSanctumAuth()

// computed properties
const shoppingLists = ref([])
const textareaValue = ref('')
const selectedListId = ref(null)
const selectedListName = ref('')



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

const debouncedSaveList = useDebounceFn(async () => {
    if (!selectedListId.value) return;
    try {
        // Get CSRF cookie
        await $fetch('http://localhost:8000/sanctum/csrf-cookie', {
            credentials: 'include'
        });

        // Get the CSRF token from the cookie
        function getCookie(name) {
            const value = `; ${document.cookie}`;
            const parts = value.split(`; ${name}=`);
            if (parts.length === 2) return decodeURIComponent(parts.pop().split(';').shift());
        }
        const xsrfToken = getCookie('XSRF-TOKEN');

        await $fetch(`http://localhost:8000/api/shopping-lists/${selectedListId.value}`, {
            method: 'PATCH',
            headers: {
                'Accept': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            body: {
                text: textareaValue.value,
                name: selectedListName.value
            },
            credentials: 'include'
        });
        // Update the local list with the new text so UI stays in sync
        const list = shoppingLists.value.find(list => list.id === selectedListId.value)
        if (list)
            list.text = textareaValue.value
        list.name = selectedListName
    } catch (error) {
        console.error('Error updating list:', error)
    }
}, 1000)

watch(textareaValue, () => {
    if (selectedListId.value) debouncedSaveList()
})

watch(selectedListName, () => {
    if (selectedListId.value) debouncedSaveList()
})

// Fetch shopping lists on mount
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

// Create new list
async function createNewList() {
    try {
        // Get CSRF cookie
        await $fetch('http://localhost:8000/sanctum/csrf-cookie', {
            credentials: 'include'
        });

        // Get the CSRF token from the cookie
        function getCookie(name) {
            const value = `; ${document.cookie}`;
            const parts = value.split(`; ${name}=`);
            if (parts.length === 2) return decodeURIComponent(parts.pop().split(';').shift());
        }
        const xsrfToken = getCookie('XSRF-TOKEN');

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

// Handle list click
function handleListClick(listId) {
    selectedListId.value = listId
    // Find the selected list directly from shoppingList
    const list = shoppingLists.value.find(list => list.id === listId)
    // set textare to the list's text (or empty if not found)
    textareaValue.value = list?.text || ''
    selectedListName.value = list?.name || 'New Shopping List'
}

// helper function to get the CSRF token from the cookie
async function getCsrfToken() {
    // Ensure the CSRF cookie is set
    await $fetch('http://localhost:8000/sanctum/csrf-cookie')

    // Get the CSRF token from the cookie
    const value = `; ${document.cookie}`;
    const parts = value.split(`; CSRF-TOKEN=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
    return null;
}

// function to truncate the list name if it is too long
function truncateListName(name) {
    if (name.length < 30) return name
    return name.slice(0, 30) + '...'
}

// function to delete list
async function deleteList(id) {
    try {
        // Get CSRF cookie
        await $fetch('http://localhost:8000/sanctum/csrf-cookie', {
            credentials: 'include'
        });

        // Get the CSRF token from the cookie
        function getCookie(name) {
            const value = `; ${document.cookie}`;
            const parts = value.split(`; ${name}=`);
            if (parts.length === 2) return decodeURIComponent(parts.pop().split(';').shift());
        }
        const xsrfToken = getCookie('XSRF-TOKEN');

        // Delete the list
        const newList = await $fetch(`http://localhost:8000/api/shopping-lists/${id}`, {
            method: 'DELETE',
            headers: {
                'Accept': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            credentials: 'include'
        });

        alert('Shopping list deleted!');
        window.location.reload();

    } catch (error) {
        console.error('Error deleting list:', error);
        alert('Error creating new list. Please try again.');
    }
}

</script>