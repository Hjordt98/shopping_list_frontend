<template>
    <div class="drawer lg:drawer-open">
        <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col items-center justify-center">
            <!-- Page content here -->
            <label for="my-drawer-2" class="btn btn-primary drawer-button lg:hidden">
                Open drawer
            </label>
        </div>
        <div class="drawer-side">
            <h1 class="text-2xl font-bold text-center text-gray-300 mb-4 mx-auto mt-4">Shopping List</h1>
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

</script>