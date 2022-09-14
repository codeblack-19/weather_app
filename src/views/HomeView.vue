<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]" v-model="searchQuery" @input="getSearchResults" name="search" id="" placeholder="Search for city or state">
      <ul v-if="searchResults" class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
        <p v-if="searchError">Sorry, something went wrong, please try again</p>
        <p v-if="!searchError && searchResults.length === 0">No results match your query, try a difference term</p>
        <template v-else>
          <li v-for="result in searchResults" :key="searchResults.id" @click="previewCity(result)" class="py-2 px-2 cursor-pointer">
            {{result.place_name}}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CitiesList />

        <template #fallback>
          <Skeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>


<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';
import CitiesList from '../components/CitiesList.vue';
import Skeleton from '../components/CityCardSkeleton.vue';

const APIKey = "pk.eyJ1IjoiY29kZXByb3BoZXQiLCJhIjoiY2w4MWViY2xrMGZkczNvdDlkbTdjZTdkZiJ9.P_suWnw4iL0zc93gPvi3gw"
const searchQuery = ref("")
const queryTimeout = ref(null)
const searchResults = ref(null)
const searchError = ref(null)
const router = useRouter()

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if(searchQuery.value !== ""){
      try{
        const results = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${APIKey}&types=place`
        )
        searchResults.value = results.data.features;
      }catch(e){
        searchError.value = true;
      }
      return;
    }
    searchResults.value = null;
  }, 300)
}

const previewCity = (result) => {
  const [city, state] = result.place_name.split(",")
  router.push({name: "cityview", params: {state: state.replaceAll(" ", ""), city: city}, query: {
    lat: result.geometry.coordinates[1],
    lng: result.geometry.coordinates[0],
    preview: true
  }})
}
</script>
