<template>
    <div>
        <Suspense>
            <AsyncCityView />

            <template #fallback>
                <p class="text-white p-2 text-center">Loading....</p>
            </template>
        </Suspense>
    </div>
</template>

<script setup>
import AsyncCityView from '../components/AsyncCityView.vue';

</script>
