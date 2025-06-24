<template>
    <div class="drawer lg:drawer-open">
        <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col min-h-screen bg-base-200">


            <!-- ---------------------------------------------------- Buttons ---------------------------------------------------- -->
            <div class="flex justify-between items-center p-6">
                <div class="flex gap-x-4">
                    <button @click="showCreateItemPopup = true" class="btn btn-ghost border-gray-300 border-2">
                        Create New Item in selected list
                    </button>

                    <button v-if="selectedListId !== 'all-favorite-items'" @click="deleteList(selectedListId)"
                        class="btn btn-ghost border-gray-300 border-2">
                        Delete selected list
                    </button>
                    <button v-if="selectedListId !== 'all-favorite-items'" @click="FavoriteList(selectedListId)"
                        class="btn btn-ghost border-gray-300 border-2">
                        {{ isFavorite ? 'Remove from favorites' : 'Add to favorites' }}
                    </button>
                </div>
                <div>
                    <button @click="handleSignOut" class="btn btn-ghost border-gray-300 border-2">
                        Sign Out
                    </button>
                </div>
            </div>


            <!-- ---------------------------------------------------- Alerts ---------------------------------------------------- -->
            <Alert class="mt-20 ml-100" :show="showDeleteSuccess" message="Shopping List Deleted!" type="success" />
            <Alert class="mt-20 ml-100" :show="showCreateSuccess" message="Shopping List Created!" type="success" />
            <Alert class="mt-20 ml-100" :show="showCreateItemSuccess" message="Item Updated!" type="success" />
            <Alert class="mt-20 ml-100" :show="showDeleteItemSuccess" message="Item Deleted!" type="success" />
            <Alert class="mt-20 ml-100" :show="showAddItemSuccess" message="Item added to list!" type="success" />
            <Alert class="mt-20 ml-100" :show="generalError" message="Something went wrong. Please try again."
                type="error" />
            <Alert class="mt-20 ml-100" :show="showFavoriteSuccess" message="List added to favorites!" type="success" />
            <Alert class="mt-20 ml-100" :show="showLastListDeleted"
                message="Last list deleted. New list was created automatically." type="success" />


            <!-- ---------------------------------------------------- List items ---------------------------------------------------- -->
            <div class="flex flex-col items-center justify-center w-full mt-10 px-4 relative">
                <p class="text-gray-400 text-sm mb-4 text-left w-full max-w-6xl ml-6">Search for an item by writing in
                    the search bar. Or filter by category by selecting a category from the dropdown menu.</p>
                <div class="flex gap-x-4 w-full max-w-6xl ml-7">
                    <label class="input">
                        <svg class="h-[1em] opacity-50" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                            <g stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" fill="none"
                                stroke="currentColor">
                                <circle cx="11" cy="11" r="8"></circle>
                                <path d="m21 21-4.3-4.3"></path>
                            </g>
                        </svg>
                        <input type="search" required placeholder="Search for an item" v-model="searchListItemInput"
                            @input="searchListItems()" />
                    </label>
                    <select v-model="filterListInput" class="select mb-4" @change="filterListByCategory()">
                        <option value="">Show All Items</option>
                        <option v-for="category in categories" :key="category.id" :value="category.id">
                            {{ category.name }}
                        </option>
                    </select>
                    <button v-if="selectedListId !== 'all-favorite-items'" @click="priotizeFavoriteItemsFirst()"
                        class="btn btn-ghost border-gray-300 border-2">
                        Priotize favorite items first
                    </button>
                </div>

                <div class="flex gap-x-4 w-full max-w-6xl ml-7 space-x-23">
                    <div>
                        <p class="text-gray-200 text-m mb-2">Total Items: {{ selectedListItems.length }}</p>
                        <p class="text-gray-200 text-m mb-2">Total price for all items: {{selectedListItems.reduce((acc,
                            item) => acc + item.price_per_unit * item.quantity, 0)}} DKK</p>
                    </div>
                    <div>
                        <p class="text-gray-200 text-m mb-2">Items left to buy: {{selectedListItems.filter(item =>
                            !item.is_checked).length}}</p>

                        <p class="text-gray-200 text-m mb-2">Total price for items left to buy: {{
                            selectedListItems.filter(item => !item.is_checked).reduce((acc, item) => acc +
                                item.price_per_unit * item.quantity, 0)}} DKK</p>
                    </div>

                </div>

                <div class="w-full max-w-6xl bg-base-200 rounded-lg p-4 shadow-md">
                    <p class="text-gray-400 text-sm mb-4">To change the name of the list, click on the list name below
                        and
                        type
                        in the new name. It will be saved automatically.</p>
                    <input v-if="selectedListId !== 'all-favorite-items'" type="text" v-model="selectedListName"
                        @blur="updateListName(selectedListId)"
                        class="w-full bg-transparent text-gray-400 hover:text-white focus:text-white focus:outline-none mb-4 text-2xl font-bold" />
                    <h2 v-else class="w-full bg-transparent text-gray-400 mb-4 text-2xl font-bold">All Favorite Items
                    </h2>

                    <!-- ---------------------------------------------------- Header row for items ----------------------------------------------- -->
                    <div class="flex items-center font-semibold text-gray-400 border-b border-gray-600 pb-2 mb-2">
                        <div class="w-20">Favorite</div>
                        <div class="w-20 ">Bought</div>
                        <div class="flex-1 pl-6">Item name</div>
                        <div class="pr-9">Category</div>
                        <div class="pr-6">Quantity</div>
                        <div class="pr-6">Price per unit</div>
                        <div class="pr-6">Total price</div>
                        <div class="pr-41">Actions</div>
                    </div>


                    <!-- ---------------------------------------------------- Items in list ---------------------------------------------------- -->
                    <div v-if="selectedListItems.length && selectedListId !== null">
                        <div v-for="item in selectedListItems" :key="item.id"
                            class="flex items-center border-b border-gray-700 last:border-b-0 py-2">
                            <div class="w-20 flex justify-start">
                                <input @click="filterByFavoriteItemsFirst(item.id)" type="checkbox" class="checkbox"
                                    :checked="item.is_favorite" />
                            </div>
                            <div class="w-16 flex justify-start">
                                <input @click="toggleItem(item.id)" type="checkbox" class="checkbox"
                                    :checked="item.is_checked" />
                            </div>
                            <div class="flex-1 pl-10 pr-30">
                                <p v-if="item.name.length < 70">
                                    {{ item.name }}
                                </p>
                                <p v-else>
                                    {{ item.name.slice(0, 70) + '...' }}
                                </p>
                            </div>
                            <div class="w-24 text-left pl-4 pr-22">
                                {{ getCategoryName(item.category_id) }}
                            </div>
                            <div class="w-24 text-left pl-4">x{{ item.quantity }}</div>
                            <div class="w-24 text-left pl-4">
                                <p> {{ item.price_per_unit }} DKK</p>
                            </div>
                            <div class="w-24 text-left pl-4">
                                <p> {{ item.price_per_unit * item.quantity }} DKK</p>
                            </div>
                            <div class="w-32 flex justify-start">
                                <button @click="deleteItem(item.id)"
                                    class="btn btn-ghost border-gray-300 border-2">Delete Item</button>
                            </div>
                            <button @click="openEditItemPopup(item)" class="btn btn-ghost border-gray-300 border-2">
                                Edit Item
                            </button>
                            <div v-if="updateItemLoading" class="w-32 flex justify-start">
                                <span class="loading loading-spinner loading-xs"></span>
                            </div>
                        </div>
                    </div>
                    <div v-else-if="selectedListId !== null && selectedListItems.length === 0"
                        class="text-gray-400 mt-4 text-center">No items in this list. Click "create new item" to add
                        items to this list.</div>
                    <div v-else class="text-gray-400 mt-4 text-center">No list selected.</div>
                </div>
            </div>


            <!-- ---------------------------------------------------- Sidebar ---------------------------------------------------- -->
        </div>
        <div class="drawer-side bg-base-300 w-80 min-h-screen">
            <h1 class="text-2xl font-bold text-center text-gray-300 mb-4 mx-auto mt-4">Shopping List</h1>
            <div class="w-full">
                <button class="btn btn-ghost mb-4 mx-auto block border-gray-300 rounded-md"
                    @click="createNewList">Create
                    new list</button>
            </div>


            <!-- ---------------------------------------------------- Today & Yesterday ---------------------------------------------------- -->
            <h1 class="text-1xl ml-3 mb-2">Today & Yesterday</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in todayAndYesterdayLists" :key="list.id" class="mb-2">
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


            <!-- ---------------------------------------------------- Older than 3 days ---------------------------------------------------- -->
            <h1 class="text-1xl ml-3 mb-2">Older than 3 days</h1>
            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in olderLists" :key="list.id">
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


            <!-- ---------------------------------------------------- Favorite  ---------------------------------------------------- -->
            <h1 class="text-1xl ml-3 mb-2">Favorite lists</h1>

            <ul class="menu bg-base-200 text-base-content w-80 p-4">
                <li v-for="list in favoriteListDefaultList" :key="list.id" class="mb-2">
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
        </div>
    </div>


    <!-- ---------------------------------------------------- Create item popup ---------------------------------------------------- -->
    <div v-if="showCreateItemPopup" class="fixed inset-0 flex items-center justify-center">
        <div class="bg-gray-800 border-4 border-gray-300 p-10 z-[10000] min-h-[200px]">
            <div class="mb-">
                <h2 v-if="selectedListName.length < 30">Creating item for: {{ selectedListName }}</h2>
                <h2 v-else>Creating item for: {{ selectedListName.slice(0, 30) + '...' }}</h2>
            </div>
            <div class="mb-8">
                <p class="text-gray-400 text-sm mb-2">Item Name</p>
                <input v-model="newItemName" placeholder="Item name" class="input input-boredered"
                    @keydown.enter="addItem" @keydown.escape="showCreateItemPopup = false" />
                <div v-if="newItemNameError" class="text-red-500 text-sm mt-1">
                    {{ newItemNameError }}
                </div>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Category (select a category from the dropdown menu)</p>
                <select v-model="newItemCategory" class="select mb-4">
                    <option disabled selected>Pick a category</option>
                    <option v-for="category in categories" :key="category.id" :value="category.id">
                        {{ category.name }}
                    </option>
                </select>
                <div v-if="newItemCategoryError" class="text-red-500 text-sm mt-1">
                    {{ newItemCategoryError }}
                </div>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Quantity (must be a whole number)</p>
                <input v-model="newItemQuantity" type="number" placeholder="Quantity, must be a whole number" step="1"
                    min="1" max="100" class="input input-bordered" />
                <div v-if="newItemQuantityError" class="text-red-500 text-sm mt-1">
                    {{ newItemQuantityError }}
                </div>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Price per unit</p>
                <input v-model="newItemPricePerUnit" type="number" placeholder="Price per unit"
                    class="input input-bordered" step="0.01" min="0" />
                <div v-if="newItemPricePerUnitError" class="text-red-500 text-sm mt-1">
                    {{ newItemPricePerUnitError }}
                </div>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Mark new item as favorite</p>
                <div class="w-20 flex justify-start">
                    <input type="checkbox" class="checkbox" v-model="newItemIsFavorite" />
                </div>
            </div>
            <div class="flex gap-x-4">
                <button @keypress.enter="addItem" @click="addItem" class="btn btn-ghost border-gray-300 border-2">Add
                    item to list</button>
                <button @click="showCreateItemPopup = false" class="btn btn-ghost border-gray-300 border-2">Stop adding
                    items to list</button>
            </div>

        </div>
        <div class="fixed inset-0 bg-black opacity-50 z-[9998]" @click="showCreateItemPopup = false"></div>
    </div>


    <!-- ---------------------------------------------------- Edit item popup ---------------------------------------------------- -->
    <div v-if="showEditItemPopup" class="fixed inset-0 flex items-center justify-center">
        <div class="bg-gray-800 border-4 border-gray-300 p-10 z-[10000] min-h-[300px] min-w-[600px]">
            <div class="mb-4">
                <h2 v-if="selectedListName.length < 30">Editing item in: {{ selectedListName }}</h2>
                <h2 v-else>Editing item in: {{ selectedListName.slice(0, 30) + '...' }}</h2>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Item Name</p>
                <input @keydown.escape="showEditItemPopup = false"
                    @keydown.enter="updateItem(editingItem.id, editingItem.name, editingItem.quantity, editingItem.is_checked, editingItemCategory, editingItem.is_favorite, editingItemPricePerUnit)"
                    v-model="editingItem.name" class="input input-boredered" />
            </div>
            <div>
                <p class="text-gray-400 text-sm mb-2">Category</p>
                <select v-model="editingItemCategory" class="select w-full mb-4">
                    <option disabled selected>Pick a category</option>
                    <option v-for="category in categories" :key="category.id" :value="category.id"
                        :default="editingItem.category_id">
                        {{ category.name }}
                    </option>
                </select>
                <div v-if="editingItemCategoryError" class="text-red-500 text-sm mt-1">
                    {{ editingItemCategoryError }}
                </div>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Quantity (must be a whole number)</p>
                <input @keydown.escape="showEditItemPopup = false"
                    @keydown.enter="updateItem(editingItem.id, editingItem.name, editingItem.quantity, editingItem.is_checked, editingItemCategory, editingItem.is_favorite, editingItemPricePerUnit)"
                    v-model="editingItem.quantity" type="number" placeholder="Quantity, must be a whole number" step="1"
                    min="1" max="100" class="input input-bordered" />
                <div v-if="editingItemQuantityError" class="text-red-500 text-sm mt-1">
                    {{ editingItemQuantityError }}
                </div>
            </div>
            <div class="mb-4">
                <p class="text-gray-400 text-sm mb-2">Price per unit</p>
                <input v-model="editingItemPricePerUnit" type="number" placeholder="Price per unit"
                    class="input input-bordered" step="0.01" min="0" />
                <!-- <div v-if="editingItemPricePerUnitError" class="text-red-500 text-sm mt-1">
                    {{ editingItemPricePerUnitError }}
                </div> -->
            </div>
            <div class="flex gap-x-4">
                <button :disabled="updateItemLoading"
                    @click="updateItem(editingItem.id, editingItem.name, editingItem.quantity, editingItem.is_checked, editingItemCategory, editingItem.is_favorite, editingItemPricePerUnit)"
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
import { isInteger, isEmpty } from 'usemods'
// ---------------------------------------------------- useSanctumAuth hook to handle authentication ----------------------------------------------------
const { logout } = useSanctumAuth()

// ---------------------------------------------------- computed properties ----------------------------------------------------
// Lists
const shoppingLists = ref([])
const selectedListId = ref(null)
const selectedListName = ref('')
const selectedListItems = ref([])

// Editing item
const editingItemId = ref(null)
const editingName = ref('')
const editingItemCategoryError = ref('')
const showEditItemPopup = ref(false)
const editingItem = ref()
const editingItemPricePerUnit = ref(0)


// Creating item
const newItemName = ref('')
const newItemQuantity = ref(1)
const newItemQuantityError = ref('')
const newItemCategoryError = ref('')
const newItemCategory = ref('')
const editingItemCategory = ref('')
const newItemIsFavorite = ref(false)
const newItemPricePerUnit = ref(0)

// Alerts
const showDeleteSuccess = ref(false)
const showCreateSuccess = ref(false)
const showCreateItemSuccess = ref(false)
const showDeleteItemSuccess = ref(false)
const showAddItemSuccess = ref(false)
const showFavoriteSuccess = ref(false)
const showLastListDeleted = ref(false)

// other
const updateItemLoading = ref(false)
const searchListItemInput = ref('')
const filterListInput = ref('')
const textareaValue = ref('')
const showCreateItemPopup = ref(false)
const categories = ref([])
const isFavorite = ref(null)
const favoriteItemsList = ref([])

// Errors
const newItemPricePerUnitError = ref('')
const newItemNameError = ref('')
const generalError = ref(false)
const editingItemQuantityError = ref('')

// ---------------------------------------------------- computed properties functions ----------------------------------------------------
const todayAndYesterdayLists = computed(() => {
    const now = new Date();
    // Get UTC timestamp for now
    const nowUTC = now.getTime() - now.getTimezoneOffset() * 60000;
    // 48 hours ago in UTC
    const twoDaysAgoUTC = nowUTC - 48 * 60 * 60 * 1000;

    return shoppingLists.value
        .filter(list => {
            if (!list.is_favorite) {
                const listUpdatedDate = new Date(list.updated_at);
                const listUTC = listUpdatedDate.getTime();
                return listUTC >= twoDaysAgoUTC && listUTC <= nowUTC;
            }
        }).sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
})

const olderLists = computed(() => {
    const threeDaysAgo = new Date()
    threeDaysAgo.setDate(threeDaysAgo.getDate() - 3)

    return shoppingLists.value
        .filter(list => {
            const listUpdatedDate = new Date(list.updated_at)
            return listUpdatedDate < threeDaysAgo
        })
        .sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
})

const favoriteLists = computed(() => {
    return shoppingLists.value.filter(list => list.is_favorite).sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
})

const favoriteListDefaultList = computed(() => {
    const allFavoriteItems = {
        id: 'all-favorite-items',
        name: 'All Favorite Items',
        items: favoriteItemsList.value
    }
    return [allFavoriteItems, ...favoriteLists.value]
})


// ---------------------------------------------------- misc ----------------------------------------------------
const debouncedSaveList = useDebounceFn(async () => {
    if (!selectedListId.value) return;
    try {
        const xsrfToken = await getCsrfToken();

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
        if (response.length === 0) {
            createNewList()
        }
        shoppingLists.value = response
        sortShoppingLists()
        handleListClick(shoppingLists.value[0].id)
        getCategories()
        handleFavoriteList()
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
        const xsrfToken = await getCsrfToken();

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

        updateLocalShoppingListsWithNewList(newList)  

        handleListClick(newList.id)

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
        const xsrfToken = await getCsrfToken();

        // Delete the list
        await $fetch(`http://localhost:8000/api/shopping-lists/${id}`, {
            method: 'DELETE',
            headers: {
                'Accept': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            credentials: 'include'
        });
        updateLocalShoppingListsWithDeletedList(id)
        selectedListId.value = null
        sortShoppingLists()

        checkIfListisEmpty()
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

// function to delete item from list
async function deleteItem(itemId) {
    try {
        const xsrfToken = await getCsrfToken();

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
        updateLocalListItems(itemId, '', '', '', '')
        showDeleteItemSuccess.value = true
        setTimeout(() => showDeleteItemSuccess.value = false, 3000)
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

// function to toggle item
async function updateItem(itemId, name, quantity, is_checked, category_id, is_favorite, editingItemPricePerUnit) {
    try {
        const xsrfToken = await getCsrfToken();

        // Update the item in the backend
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
                is_checked: is_checked,
                category_id: category_id,
                is_favorite: is_favorite,
                price_per_unit: editingItemPricePerUnit
            },
            credentials: 'include'
        });

        updateLocalListItems(itemId, name, quantity, category_id, is_favorite, editingItemPricePerUnit)

        updateLocalShoppingLists(itemId, name, quantity, category_id, is_checked, is_favorite)

        sortSelectedListItemsByCategory()
        
        handleFavoriteList();

        showCreateItemSuccess.value = true;
        showEditItemPopup.value = false;
        setTimeout(() => showCreateItemSuccess.value = false, 3000);
    } catch (error) {
        generalError.value = true;
        setTimeout(() => generalError.value = false, 3000);
    }
}

async function updateList(listId, updates) {
    try {
        const xsrfToken = await getCsrfToken();

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

        updateLocalShoppingListsWithUpdatedList(listId, updates)
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}

async function createNewItem() {
    try {
        const xsrfToken = await getCsrfToken();

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
                shopping_list_id: selectedListId.value,
                category_id: newItemCategory.value,
                is_favorite: newItemIsFavorite.value,
                price_per_unit: newItemPricePerUnit.value
            },
            credentials: 'include'
        })
        updateLocalShoppingListsWithNewItem(newItem)

        clearForm()
        showAddItemSuccess.value = true
        setTimeout(() => showAddItemSuccess.value = false, 3000)
        handleFavoriteList()
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}


// get all categories
async function getCategories() {
    try {
        const xsrfToken = await getCsrfToken();

        const response = await $fetch('http://localhost:8000/api/categories', {
            headers: {
                'Accept': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            credentials: 'include'
        })
        categories.value = response
        console.log(categories)
    } catch (error) {
        generalError.value = true
        setTimeout(() => generalError.value = false, 3000)
    }
}


// update list favorite
async function updateListFavorite(listId) {
    try {
        const xsrfToken = await getCsrfToken();

        await $fetch(`http://localhost:8000/api/shopping-lists/${listId}/favorite`, {
            method: 'PATCH',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'X-XSRF-TOKEN': xsrfToken
            },
            body: {
                favorite: isFavorite.value
            },
            credentials: 'include'
        })
        const listIndex = shoppingLists.value.findIndex(list => list.id === listId)
        if (listIndex !== -1) {
            // First set the favorite status in the shopping lists array to keep data in sync
            shoppingLists.value[listIndex].favorite = isFavorite.value
            // Then set isFavorite to match the array value to ensure UI state matches data
            // This double assignment helps prevent any edge cases where the values could get out of sync
            isFavorite.value = shoppingLists.value[listIndex].favorite
            window.location.reload()
        }
        showFavoriteSuccess.value = true
        setTimeout(() => showFavoriteSuccess.value = false, 3000)
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
    if (listId === 'all-favorite-items') {
        selectedListId.value = 'all-favorite-items'
        selectedListItems.value = favoriteItemsList.value
        handleFavoriteList()
        sortSelectedListItemsByCategory()
        return
    }

    const list = shoppingLists.value.find(list => list.id === listId)
    selectedListName.value = list?.name || 'New Shopping List'
    selectedListItems.value = list?.items || []
    isFavorite.value = list?.is_favorite || false
    selectedListId.value = listId
    handleFavoriteList()
    sortSelectedListItemsByCategory()
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
        updateItem(itemId, item.name, item.quantity, !item.is_checked, item.category_id, item.is_favorite);
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
    if (!isInteger(newItemQuantity.value)) {
        newItemQuantityError.value = 'Quantity must be a whole number. Please enter a valid quantity.'
        return
    }

    if (isEmpty(newItemCategory.value)) {
        newItemCategoryError.value = 'Category is required. Please pick a category.'
        return
    }

    if (isEmpty(newItemPricePerUnit.value)) {
        newItemPricePerUnitError.value = 'Value can not be empty and must be a positive number or 0.'
        return
    }

    const priceStr = newItemPricePerUnit.value.toString()
    if (!/^\d+(\.\d{1,2})?$/.test(priceStr)) {
        newItemPricePerUnitError.value = 'Price per unit can have maximum 2 decimal places.'
        return
    }

    if (isEmpty(newItemName.value)) {
        newItemNameError.value = 'Item name is required. Please enter a name for the item.'
        return
    }
    shoppingLists.value.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
    createNewItem()

}


// Open edit item popup
function openEditItemPopup(item) {
    editingItem.value = item
    showEditItemPopup.value = true
    editingItemCategory.value = item.category_id
}


// Get CSRF token
async function getCsrfToken() {
    // Ensure the CSRF cookie is set
    await $fetch('http://localhost:8000/sanctum/csrf-cookie')

    // Get the CSRF token from the cookie
    const value = `; ${document.cookie}`;
    const parts = value.split(`; XSRF-TOKEN=`);
    if (parts.length === 2) {
        return decodeURIComponent(parts.pop().split(';').shift());
    }
    throw new Error('CSRF token not found');
}

function getCategoryName(categoryId) {
    // Use == for loose comparison (string/number), or convert both to Number
    const cat = categories.value.find(c => c.id === categoryId)
    return cat ? cat.name : 'Unknown'
}

function searchListItems() {
    const searchItem = searchListItemInput.value.toLowerCase()
    filterListInput.value = ''

    if (searchItem.length > 0) {
        selectedListItems.value = selectedListItems.value.filter(item => item.name.toLowerCase().includes(searchItem))
    } else {
        selectedListItems.value = shoppingLists.value.find(list => list.id === selectedListId.value)?.items || []
        searchListItemInput.value = ''
    }
}

function filterListByCategory() {
    const selectedCategoryId = filterListInput.value
    searchListItemInput.value = ''

    // we check if category is s
    if (selectedCategoryId !== '') {
        const originalItems = shoppingLists.value.find(list => list.id === selectedListId.value)?.items
        selectedListItems.value = originalItems.filter(item => item.category_id === parseInt(selectedCategoryId))
    } else {
        selectedListItems.value = shoppingLists.value.find(list => list.id === selectedListId.value)?.items
    }
}

// Toggle item
async function filterByFavoriteItemsFirst(itemId) {
    const item = selectedListItems.value.find(item => item.id === itemId);
    if (item) {
        await updateItem(itemId, item.name, item.quantity, item.price_per_unit, item.is_checked, item.category_id, !item.is_favorite);
        handleFavoriteList()
    }
}

function priotizeFavoriteItemsFirst() {
    searchListItemInput.value = ''
    filterListInput.value = ''
    selectedListItems.value = selectedListItems.value.sort((a, b) => b.is_favorite - a.is_favorite)
}

function handleFavoriteList() {
    // Clear the array first!
    favoriteItemsList.value = [];
    shoppingLists.value.forEach(list => {
        if (list.items) {
            list.items.forEach(item => {
                if (item.is_favorite) {
                    favoriteItemsList.value.push(item)
                }
            })
        }
    })
}

function sortSelectedListItemsByCategory() {
    selectedListItems.value = selectedListItems.value.sort((a, b) => a.category_id - b.category_id)
}

function updateLocalListItems(itemId, name, quantity, category_id, is_favorite, price_per_unit) {

    // Update local state in selectedListItems
    const itemIndex = selectedListItems.value.findIndex(item => item.id === itemId);
    if (itemIndex !== -1) {
        selectedListItems.value[itemIndex] = {
            ...selectedListItems.value[itemIndex],
            name: name,
            quantity: quantity,
            category_id: category_id,
            is_favorite: is_favorite,
            price_per_unit: price_per_unit
        };
    }
}

function updateLocalShoppingLists(itemId, name, quantity, category_id, is_checked, is_favorite) {

    // Update the item in the main shoppingLists array
    for (const list of shoppingLists.value) {
        const idx = list.items.findIndex(item => item.id === itemId);
        if (idx !== -1) {
            list.items[idx] = {
                ...list.items[idx],
                name: name,
                quantity: quantity,
                category_id: category_id,
                is_checked: is_checked,
                is_favorite: is_favorite
            };
            break;
        }
    }
}

function clearForm() {
    newItemName.value = ''
    newItemQuantity.value = ''
    newItemCategory.value = ''
    newItemIsFavorite.value = false
    newItemPricePerUnit.value = ''
}

function updateLocalShoppingListsWithNewItem(newItem) {
    //updaet the local list with the new item
    const listIndex = shoppingLists.value.findIndex(list => list.id === selectedListId.value)
    if (listIndex !== -1) {
        shoppingLists.value[listIndex].items.push(newItem)
    }
}

function updateLocalShoppingListsWithNewList(newList) {
    shoppingLists.value.push(newList)
}

function updateLocalShoppingListsWithUpdatedList(listId, updates) {
    // Update local state
    const listIndex = shoppingLists.value.findIndex(list => list.id === listId);
    if (listIndex !== -1) {
        shoppingLists.value[listIndex] = { ...shoppingLists.value[listIndex], ...updates };
    }
}

function updateLocalShoppingListsWithDeletedList(listId) {
    shoppingLists.value = shoppingLists.value.filter(list => list.id !== listId)
}

function sortShoppingLists() {
    shoppingLists.value.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
}

async function checkIfListisEmpty() {
    if (
            shoppingLists.value.length === 0 &&
            olderLists.value.length === 0 &&
            favoriteLists.value.length === 0
        ) {
            await createNewList()
            showLastListDeleted.value = true
            setTimeout(() => showLastListDeleted.value = false, 3000)
        } else if (shoppingLists.value.length > 0) {
            showDeleteSuccess.value = true
            setTimeout(() => showDeleteSuccess.value = false, 3000)
        }
}


</script>