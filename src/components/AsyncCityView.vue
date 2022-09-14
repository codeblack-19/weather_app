<template>
    <div class="flex flex-col flex-1 items-center">
        <!-- Banner -->
        <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
            <p>
                You are currently previewing this city, click the "+" icon to start tracking this city.
            </p>
        </div>

        <!-- weather overview -->
        <div class="flex flex-col items-center text-white py-12">
            <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
            <p class="text-sm mb-12">
                {{
                    new Date(weatherData.current_weather.time).toLocaleDateString(
                        "en-uk",
                        {
                            weekday: "short",
                            day: "2-digit",
                            month: 'long'
                        }
                    )
                }}
                {{
                    new Date(weatherData.current_weather.time).toLocaleTimeString(
                        "en-us",
                        {
                            timeStyle: 'short',
                        }
                    )
                }}
            </p>
            <p class="text-8xl mb-8">
                {{ Math.round(weatherData.current_weather.temperature) }}&deg;
            </p>
            <div class="text-center">
                <p>
                    Feels Like
                    {{ Math.round(weatherData.daily.temperature_2m_max[3]) }}&deg;
                </p>
                <p class="capitalize">
                   Snow showers slight
                </p>
            </div>
        </div>

        <hr class="border-white border-opacity-10 border w-full" /> 

        <!-- hourly -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white" >
                <h2 class="mb-4">Hourly Weather</h2>
                <div class="flex gap-10 overflow-x-scroll">
                    <div v-for="(hourData, index) in weatherData.newObjhourly" :key="index" class="flex flex-col gap-4 items-center">
                        <p class="whitespace-nowrap text-md mb-4">
                            {{
                                new Date(hourData.time).toLocaleTimeString("en-us", {
                                    hour: "numeric"
                                })
                            }}
                        </p>
                        <img 
                            class="w-auto h-[50px] object-cover"
                            :src="
                            `http://openweathermap.org/img/wn/0${hourData.code > 45 || hourData.code === 0 ? 3 : hourData.code}d@2x.png`
                            "
                            alt=""
                        />
                        <p class="text-xl mb-3">
                            {{ Math.round(hourData.temp) }}&deg;
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- Weakly Weather -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4">7 Day Forcast</h2>
                <div class="flex items-center" v-for="(day, index) in weatherData.newObjDaily" :key="index">
                    <p class="flex-1">
                        {{
                            new Date(day.time).toLocaleDateString("en-us", { 
                                weekday: 'long'
                            })
                        }}
                    </p>
                    <img 
                        class="w-[50px] h-[50px] object-cover"
                        :src="
                            `http://openweathermap.org/img/wn/0${day.code > 45 || day.code === 0 ? 4 : day.code}d@2x.png`
                        "
                        
                        alt=""
                    />
                    <div class="flex gap-2 flex-1 justify-end">
                        <p>H: {{ Math.round(day.temp.max) }}</p>
                        <p>H: {{ Math.round(day.temp.min) }}</p>
                    </div>
                </div>
            </div>
        </div>

        <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500" @click="removeCity" v-if="cities">
            <i class="fa-solid fa-trash"></i>
            <p>Remove City</p>
        </div>
    </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter()
const cities = JSON.parse(localStorage.getItem('savedCities'));

const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(
            `https://api.open-meteo.com/v1/forecast?latitude=${route.query.lat}&longitude=${route.query.lng}&hourly=temperature_2m,apparent_temperature,weathercode&daily=weathercode,temperature_2m_max,temperature_2m_min&current_weather=true&timezone=auto`
        );

        weatherData.data.newObjhourly = []
        weatherData.data.hourly.temperature_2m.map((temp, index) => {
            return weatherData.data.newObjhourly.push({ temp: temp, time: weatherData.data.hourly.time[index], code: weatherData.data.hourly.weathercode[index], })
        })

        weatherData.data.newObjDaily = []
        weatherData.data.daily.temperature_2m_max.map((temp, index) => {
            return weatherData.data.newObjDaily.push({ 
                temp: {
                    max: temp,
                    min: weatherData.data.daily.temperature_2m_min[index]
                }, 
                time: weatherData.data.daily.time[index], 
                code: weatherData.data.daily.weathercode[index] 
            })
        })

        // // // cal current date & time
        // const localOffset = new Date().getTimezoneOffset() * 60000;
        // const utc = weatherData.data.current_weather.time * 1000 + localOffset;
        // weatherData.data.current_weather.time = utc + 1000 * weatherData.data.utc_offset_seconds;

        // // // cal hourly weather offset
        // weatherData.data.hourly.time.forEach(time => {
        //     const utc = time * 1000 + localOffset;
        //     time = utc + 1000 * weatherData.data.utc_offset_seconds;
        // });

        return weatherData.data;
    } catch (e) {
        console.log(e)
    }
}

const weatherData = await getWeatherData()

const removeCity = () => {
    if(cities){
        const updatedCites = cities.filter((city) => {
            city.id !== route.query.id
        });

        localStorage.setItem('savedCities', JSON.stringify(updatedCites));
        router.push({
            name: "home"
        })
    }
}
</script>
