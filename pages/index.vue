<template>
    <div class="drawer lg:drawer-open">
        <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col items-center justify-center">

            <!-- ---------------------------------------------------- Buttons ---------------------------------------------------- -->
            <div class="flex gap-x-4 mt-6 mr-10 space-x-">
                <div class="flex gap-x-4 justify-start">
                    <button @click="showCreateItemPopup = true" class="btn btn-ghost border-gray-300 border-2">
                        Create New Item in selected list
                    </button>

                    <button @click="deleteList(selectedListId)" class="btn btn-ghost border-gray-300 border-2">Delete
                        selected list</button>
                    <button @click="toggleFavorit(selectedListId)" class="btn btn-ghost border-gray-300 border-2">
                        Add list to favorites
                    </button>
                </div>
                <div>
                    <button @click="handleSignOut" class="btn btn-ghost border-gray-300 border-2">
                        Sign Out
                    </button>
                </div>
                <!-- ---------------------------------------------------- Alerts ---------------------------------------------------- -->
                <Alert class="mt-20 align-center" :show="showDeleteSuccess" message="Shopping List Deleted!"
                    type="success" />
                <Alert class="mt-20 align-center" :show="showCreateSuccess" message="Shopping List Created!"
                    type="success" />
                <Alert class="mt-20 align-center" :show="showCreateItemSuccess" message="Item Updated!"
                    type="success" />
                <Alert class="mt-20 align-center" :show="showDeleteItemSuccess" message="Item Deleted!"
                    type="success" />
                <Alert class="mt-20 align-center" :show="showAddItemSuccess" message="Item added to list!"
                    type="success" />
                <Alert class="mt-20 align-center" :show="generalError" message="Something went wrong. Please try again."
                    type="error" />
            </div>

            <!-- ---------------------------------------------------- No list selected ---------------------------------------------------- -->
            <div v-if="selectedListId === null">
                <h1>please select a list before creating an item</h1>
            </div>

            <!-- ---------------------------------------------------- List items ---------------------------------------------------- -->
            <div class="flex flex-col items-center justify-center w-full mt-10 px-4 relative">


                <div class="w-full max-w-6xl bg-base-200 rounded-lg p-4 shadow-md">
                    <input type="text" v-model="selectedListName" @blur="updateListName(selectedListId)"
                        class="w-full bg-transparent text-gray-400 hover:text-white focus:text-white focus:outline-none mb-4 text-2xl font-bold" />

                    <!-- ---------------------------------------------------- Header row for items ----------------------------------------------- -->
                    <div class="flex items-center font-semibold text-gray-400 border-b border-gray-600 pb-2 mb-2">
                        <div class="w-12 flex justify-start">Marked</div>
                        <div class="flex-1 pl-8">Item name</div>
                        <div class="w-24 text-left pr-45">Quantity</div>
                        <div class="w-32"></div>
                    </div>

                    <!-- ---------------------------------------------------- Items in list ---------------------------------------------------- -->
                    <div v-if="selectedListItems.length && selectedListId !== null">
                        <div v-for="item in selectedListItems" :key="item.id"
                            class="flex items-center border-b border-gray-700 last:border-b-0 py-2">
                            <div class="w-12 flex justify-start">
                                <input @click="toggleItem(item.id)" type="checkbox" class="checkbox"
                                    :checked="item.is_checked" />
                            </div>
                            <div class="flex-1 pl-8 pr-30">
                                <p v-if="item.name.length < 70">
                                    {{ item.name }}
                                </p>
                                <p v-else>
                                    {{ item.name.slice(0, 70) + '...' }}
                                </p>
                            </div>
                            <div class="w-24 text-left pl-4">x{{ item.quantity }}</div>
                            <div class="w-32 flex justify-start">
                                <button @click="deleteItem(item.id)"
                                    class="btn btn-ghost border-gray-300 border-2">Delete Item</button>
                            </div>
                            <button @click="openEditItemPopup(item)" class="btn btn-ghost border-gray-300 border-2">
                                Edit Item
                            </button>
                        </div>
                    </div>
                    <div v-else-if="selectedListId !== null && selectedListItems.length === 0"
                        class="text-gray-400 mt-4 text-center">No items in this list.</div>
                    <div v-else class="text-gray-400 mt-4 text-center">No list selected.</div>
                </div>
            </div>

            <!-- ---------------------------------------------------- Sidebar ---------------------------------------------------- -->
        </div>
        <div class=" drawer-side">
            <h1 class="text-2xl font-bold text-center text-gray-300 mb-4 mx-auto mt-4">Shopping List</h1>
            <div class="w-full">
                <button class="btn btn-ghost mb-4 mx-auto block border-gray-300 rounded-md"
                    @click="createNewList">Create
                    new list</button>
            </div>

            <h1 class="text-1xl ml-3 mb-2">Today & Yesterday</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in todayAndYesterday" :key="list.id" class="mb-2">
                    <a @click="handleListClick(list.id)" class="border-2 border-transparent block px-3 py-1"
                        :class="{ 'border-white rounded-md': selectedListId === list.id }">
                        <p v-if="list.name.length < 30">
                            {{ list.name }}
                        </p>
                        <p v-else>
                            {{ list.name.slice(0, 30) + '...' }}
                        </p>
                    </a>
                </li>
            </ul>

            <h1 class="text-1xl ml-3 mb-2">Older than 3 days</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in olderList" :key="list.id">
                    <a @click="handleListClick(list.id)"
                        :class="{ 'border-2 border-gray-300 rounded-md': selectedListId === list.id }">
                        <p v-if="list.name.length < 30">
                            {{ list.name }}
                        </p>
                        <p v-else>
                            {{ list.name.slice(0, 30) + '...' }}
                        </p>
                    </a>
                </li>
            </ul>

            <h1 class="text-1xl ml-3 mb-2">Favorite lists</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
            </ul>
        </div>
    </div>

    <!-- ---------------------------------------------------- Create item popup ---------------------------------------------------- -->
    <div v-if="showCreateItemPopup" class="fixed inset-0 flex items-center justify-center">
        <div class="bg-gray-800 border-4 border-gray-300 p-10 z-[10000] min-h-[200px]">
            <div class="mb-4">
                <h2 v-if="selectedListName.length < 30">Creating item for: {{ selectedListName }}</h2>
                <h2 v-else>Creating item for: {{ selectedListName.slice(0, 30) + '...' }}</h2>
            </div>
            <div class="mb-4">
                <input v-model="newItemName" placeholder="Item name" class="input input-boredered w-full" />
                <div v-if="newItemNameError" class="text-red-500 text-sm mt-1">
                    {{ newItemNameError }}
                </div>
            </div>
            <div class="mb-4">
                <input v-model="newItemQuantity" type="number" placeholder="Quantity, must be a whole number" step="1"
                    min="1" max="100" class="input input-bordered w-full" />
                <div v-if="newItemQuantityError" class="text-red-500 text-sm mt-1">
                    {{ newItemQuantityError }}
                </div>
            </div>
            <div class="flex gap-x-4">
                <button @click="addItem" class="btn btn-ghost border-gray-300 border-2">Add item to list</button>
                <button @click="showCreateItemPopup = false" class="btn btn-ghost border-gray-300 border-2">Stop adding
                    items to list</button>
            </div>
        </div>
        <div class="fixed inset-0 bg-black opacity-50 z-[9998]" @click="showCreateItemPopup = false"></div>
    </div>

    <!-- ---------------------------------------------------- Edit item popup ---------------------------------------------------- -->
    <div v-if="showEditItemPopup" class="fixed inset-0 flex items-center justify-center">
        <div class="bg-gray-800 border-4 border-gray-300 p-10 z-[10000] min-h-[200px]">
            <div class="mb-4">
                <h2 v-if="selectedListName.length < 30">Editing item in {{ selectedListName }}</h2>
                <h2 v-else>Editing item for: {{ selectedListName.slice(0, 30) + '...' }}</h2>
            </div>
            <div class="mb-4">
                <input v-model="editingItem.name" placeholder="Item name" class="input input-boredered w-full" />
            </div>
            <div class="mb-4">
                <input v-model="editingItem.quantity" type="number" placeholder="Quantity, must be a whole number"
                    step="1" min="1" max="100" class="input input-bordered w-full" />
                <div v-if="editingItemQuantityError" class="text-red-500 text-sm mt-1">
                    {{ editingItemQuantityError }}
                </div>
            </div>
            <div class="flex gap-x-4">
                <button @click="updateItem(editingItem.id, editingItem.name, editingItem.quantity)"
                    class="btn btn-ghost border-gray-300 border-2">Update item</button>
                <button @click="showEditItemPopup = false" class="btn btn-ghost border-gray-300 border-2">Cancel
                    editing item</button>
            </div>
        </div>
        <div class="fixed inset-0 bg-black opacity-50 z-[9998]" @click="showCreateItemPopup = false"></div>
    </div>
</template>

<script setup>

// ---------------------------------------------------- imports ----------------------------------------------------
import Alert from '~/components/alerts/Alert.vue'
import { ref, onMounted, computed, watch } from 'vue'
import { useDebounceFn } from '@vueuse/core'
import { isInteger } from 'usemods'
import { isEmpty } from 'usemods'

// ---------------------------------------------------- useSanctumAuth hook to handle authentication ----------------------------------------------------
const { logout } = useSanctumAuth()

// ---------------------------------------------------- computed properties ----------------------------------------------------
// Lists and items
const shoppingLists = ref([])
const textareaValue = ref('')
const selectedListId = ref(null)
const selectedListName = ref('')
const selectedListItems = ref([])
const showCreateItemPopup = ref(false)
const editingItemId = ref(null)
const editingName = ref('')
const newItemName = ref('')
const newItemQuantity = ref(1)
const newItemQuantityError = ref('')
const showEditItemPopup = ref(false)
const editingItem = ref()

// Alerts
const showDeleteSuccess = ref(false)
const showCreateSuccess = ref(false)
const editingItemQuantityError = ref('')
const newItemNameError = ref('')
const showCreateItemSuccess = ref(false)
const showDeleteItemSuccess = ref(false)
const showAddItemSuccess = ref(false)
const generalError = ref(false)

// ---------------------------------------------------- computed properties functions ----------------------------------------------------
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

// ---------------------------------------------------- misc ----------------------------------------------------
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
        const listIndex = shoppingLists.value.findIndex(list => list.id === selectedListId.value)
        if (listIndex !== -1) {
            shoppingLists.value[listIndex].text = textareaValue.value
            shoppingLists.value[listIndex].name = selectedListName.value
            // Move the updated list to the top
            const [updatedList] = shoppingLists.value.splice(listIndex, 1)
            shoppingLists.value.unshift(updatedList)
        }
    } catch (error) {
        console.error('Error updating list:', error)
    }
}, 1000)

// ---------------------------------------------------- watch ----------------------------------------------------
watch(textareaValue, () => {
    if (selectedListId.value) debouncedSaveList()
})

watch(selectedListName, () => {
    if (selectedListId.value) debouncedSaveList()
})

// ---------------------------------------------------- onMounted ----------------------------------------------------
onMounted(async () => {
    try {
        const response = await $fetch('http://localhost:8000/api/shopping-lists', {
            credentials: 'include'
        })
        shoppingLists.value = response
        shoppingLists.value.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
        handleListClick(shoppingLists.value[0].id)
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
})

// ---------------------------------------------------- functions ----------------------------------------------------
// ---------------------------------------------------- CRUD functions ----------------------------------------------------
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

        shoppingLists.value.push(newList)
        showCreateSuccess.value = true
        setTimeout(() => showCreateSuccess.value = false, 3000) // Hide after 3 seconds

    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

// Delete list
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
        shoppingLists.value = shoppingLists.value.filter(list => list.id !== id)
        showDeleteSuccess.value = true
        selectedListId.value = null
        setTimeout(() => showDeleteSuccess.value = false, 3000) // Hide after 3 seconds
        shoppingLists.value.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
        handleListClick(shoppingLists.value[0].id)

    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

// function to delete item from list
async function deleteItem(itemId) {
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

        // Delete the item
        await $fetch(`http://localhost:8000/api/shopping-list-items/${itemId}`, {
            method: 'DELETE',
            headers: {
                'Accept': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            credentials: 'include'
        })
        // Update the local list with the new items
        selectedListItems.value = selectedListItems.value.filter(item => item.id !== itemId)
        showDeleteItemSuccess.value = true
        setTimeout(() => showDeleteItemSuccess.value = false, 3000)
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

// function to toggle item
async function updateItem(itemId, name, quantity, is_checked) {
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

        // Update the item
        await $fetch(`http://localhost:8000/api/shopping-list-items/${itemId}`, {
            method: 'PATCH',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            body: {
                name: name,
                quantity: quantity,
                is_checked: is_checked
            },
            credentials: 'include'
        });

        // Update local state
        const itemIndex = selectedListItems.value.findIndex(item => item.id === itemId);
        if (itemIndex !== -1) {
            selectedListItems.value[itemIndex] = { ...selectedListItems.value[itemIndex], name: name, quantity: quantity };
        }
        showCreateItemSuccess.value = true
        showEditItemPopup.value = false
        setTimeout(() => showCreateItemSuccess.value = false, 3000) // Hide after 3 seconds
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

async function updateList(listId, updates) {
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

        // Update the item
        await $fetch(`http://localhost:8000/api/shopping-lists/${listId}`, {
            method: 'PATCH',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            body: updates,
            credentials: 'include'
        });

        // Update local state
        const listIndex = shoppingLists.value.findIndex(list => list.id === listId);
        if (listIndex !== -1) {
            shoppingLists.value[listIndex] = { ...shoppingLists.value[listIndex], ...updates };
        }
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

async function createNewItem() {
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

        // create the item
        const newItem = await $fetch('http://localhost:8000/api/shopping-list-items', {
            method: 'POST',
            headers: {
                'accept': 'application/json',
                'x-xsrf-token': xsrfToken
            },
            body: {
                name: newItemName.value,
                quantity: newItemQuantity.value,
                shopping_list_id: selectedListId.value
            },
            credentials: 'include'
        })
        //updaet the local list with the new item
        const listIndex = shoppingLists.value.findIndex(list => list.id === selectedListId.value)
        if (listIndex !== -1) {
            shoppingLists.value[listIndex].items.push(newItem)
        }

        //clear the form
        newItemName.value = ''
        showAddItemSuccess.value = true
        setTimeout(() => showAddItemSuccess.value = false, 3000)
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

// ---------------------------------------------------- other functions ----------------------------------------------------
async function handleSignOut() {
    try {
        await logout()
        navigateTo('/auth/login')
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}


// Handle list click
function handleListClick(listId) {
    selectedListId.value = listId
    const list = shoppingLists.value.find(list => list.id === listId)
    selectedListName.value = list?.name || 'New Shopping List'
    selectedListItems.value = list?.items || []
}


// Cancel editing
function cancelEditing() {
    editingItemId.value = null;
    editingName.value = '';
}


// Toggle item
function toggleItem(itemId) {
    const item = selectedListItems.value.find(item => item.id === itemId);
    if (item) {
        updateItem(itemId, item.name, item.quantity, !item.is_checked);
    }
}


// Update list name
async function updateListName(listId) {
    try {
        await updateList(listId, { name: selectedListName.value.trim() })
        cancelEditing();
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}


// Add item
function addItem() {
    if (!validateQuantity(newItemQuantity.value, newItemQuantityError))
        return

    if (isEmpty(newItemName.value)) {
        newItemNameError.value = 'Item name is required and must not be empty'
        return
    }
    shoppingLists.value.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
    createNewItem()

}


// Open edit item popup
function openEditItemPopup(item) {
    editingItem.value = item
    showEditItemPopup.value = true
}

// validate quantity
function validateQuantity(quantity, errorRef) {
    if (!isInteger(quantity)) {
        errorRef.value = 'Quantity must be a whole number'
        return false
    }
    if (quantity <= 0) {
        errorRef.value = 'Quantity must be greater than 0'
        return false
    }
    return true
}

// Get CSRF token
async function getCsrfToken() {
    // Ensure the CSRF cookie is set
    await $fetch('http://localhost:8000/sanctum/csrf-cookie')

    // Get the CSRF token from the cookie
    const value = `; ${document.cookie}`;
    const parts = value.split(`; CSRF-TOKEN=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
    return null;
}

</script>