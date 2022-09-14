<template>
    <div v-for="city in savedCites" :key="city.di">
        <CityCard :city="city" @click="goToCityView(city)" />
    </div>
    <p v-if="savedCites.length === 0">
        No locations add. To start tracking location, search in the field above.
    </p>
</template>

<script setup>
    import axios from 'axios';
    import { ref } from 'vue';
    import { useRouter } from 'vue-router';
    import CityCard from './CityCard.vue';

    const savedCites = ref([])
    const router = useRouter()

    const getCities = async () => {
        if(localStorage.getItem('savedCities')){
            savedCites.value = JSON.parse(localStorage.getItem('savedCities'))
        }

        const requests = []
        savedCites.value.forEach((city) => {
            requests.push(
                axios.get(
                    `https://api.open-meteo.com/v1/forecast?latitude=${city.coords.lat}&longitude=${city.coords.lng}&hourly=temperature_2m,apparent_temperature,weathercode&daily=weathercode,temperature_2m_max,temperature_2m_min&current_weather=true&timezone=auto`
                )
            )
        }); 

        const weatherData = await Promise.all(requests);

        // flicker delay
        await new Promise((res) => setTimeout(res, 1000))

        weatherData.forEach((value, index) => {
            savedCites.value[index].weather = value.data;
        })
    }

    await getCities();

    const goToCityView = (city) => {
        router.push({
            name: "cityview",
            params: {
                state: city.state,
                city: city.city
            },
            query: {
                lng: city.coords.lng,
                lat: city.coords.lat,
                id: city.id
            }
        })
    }
</script>