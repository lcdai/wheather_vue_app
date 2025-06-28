<template>
  <div class="flex flex-col flex-1 items-center">
    <div v-if="route.query.preview" class="text-white bg-weather-secondary w-full text-center">
      <p>Click to start tracking the city</p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString('en-us', {
            weekday: 'short',
            year: 'numeric',
            month: 'long',
            day: '2-digit',
          })
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString('en-us', {
            timeStyle: 'short',
          })
        }}
      </p>
      <p class="text-sm mb-12">
        {{ Math.round(weatherData.current.temp) }}°C
      </p>
      <p>
        Feel like {{ Math.round(weatherData.current.feels_like) }}°C
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        :src="`https://open-meteo.com/static/images/weather/${weatherData.current.weather[0].icon}.png`"
        :alt="weatherData.current.weather[0].description"
        class="mx-auto"
      />
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly Weather -->
    <div class="flex flex-col items-center text-white py-12">
      <h2 class="text-2xl mb-4">Hourly Forecast</h2>
      <div class="grid grid-cols-4 gap-4">
        <div v-for="(hour, index) in weatherData.hourly.time" :key="index" class="text-center">
          <p>
            {{
              new Date(hour.currentTime).toLocaleTimeString('en-us', {
                timeStyle: 'short',
              })
            }}
          </p>
          <img
            :src="`https://open-meteo.com/static/images/weather/${weatherData.hourly.weathercode[index]}.png`"
            :alt="weatherData.hourly.weathercode[index]"
            class="mx-auto"
          />
          <p>{{ Math.round(weatherData.hourly.temperature_2m[index]) }}°C</p>
          <p>{{ weatherData.hourly.weathercode[index] }}</p>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div class="flex flex-col items-center text-white py-12">
      <h2 class="text-2xl mb-4">Weekly Forecast</h2>
      <div class="grid grid-cols-7 gap-4">
        <div v-for="(day, index) in weatherData.daily.time" :key="index" class="text-center">
          <p>
            {{
              new Date(day.currentTime).toLocaleDateString('en-us', {
                weekday: 'short',
                month: 'short',
                day: '2-digit',
              })
            }}
          </p>
          <img
            :src="`https://open-meteo.com/static/images/weather/${weatherData.daily.weathercode[index]}.png`"
            :alt="weatherData.daily.weathercode[index]"
            class="mx-auto"
          />
          <p>{{ Math.round(weatherData.daily.temperature_2m_max[index]) }}°C</p>
          <p>{{ Math.round(weatherData.daily.temperature_2m_min[index]) }}°C</p>
          <p class="capitalize">
            {{ weatherData.daily.weathercode[index] }}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute } from 'vue-router';

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = axios.get(
      `https://api.open-meteo.com/v1/forecast?latitude=${route.query.lat}&longitude=${route.query.lng}&hourly=temperature_2m,precipitation_sum,weathercode&timezone=America%2FSao_Paulo`
    );

    //calculate current date and time
    const localOffset = new Date().getTimezoneOffset() * 60 * 1000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime = utc + 1000 * weatherData.data.timezone_offset;

    //calc hourly weather offset
    weatherData.data.hourly.time.map((time) => {
      const utc = time.dt * 1000 + localOffset;
      time.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });
    return weatherData.data;
  } catch (error) {
    console.error('Error fetching weather data:', error);
  }
}
const weatherData = await getWeatherData();
console.log(weatherData);
</script>
