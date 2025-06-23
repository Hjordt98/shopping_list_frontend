<template>
    <div v-if="show" role="alert" :class="[
        'alert absolute left-1/2 -translate-x-1/2 -top-15 z-50 scale-125 border-2 border-gray-300',
        `alert-${type}`
    ]">
        <!-- Success icon -->
        <svg v-if="type === 'success'" xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 shrink-0 stroke-current" fill="none" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        
        <!-- Error icon -->
        <svg v-else-if="type === 'error'" xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 shrink-0 stroke-current" fill="none" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        
        <span>{{ message }}</span>
    </div>
</template>

<script setup>
defineProps({
    show: {
        type: Boolean,
        required: true
    },
    message: {
        type: String,
        required: true
    },
    type: {
        type: String,
        required: true,
        validator: (value) => ['success', 'warning', 'error'].includes(value)
    }
})
</script>