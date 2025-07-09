<template>
  <div>
    <div v-for="city in savedCities" :key="city.id">
      <CityCard :city="city" @click="goToCityView(city)" />
    </div>

    <p v-if="savedCities.length === 0" class="text-center text-gray-500">
      No saved cities found.
    </p>
  </div>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
  }

  const requests = [];
  savedCities.value.forEach((city) => {
    requests.push(
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?q=${city.city},${city.state}&appid=YOUR_API_KEY&units=metric`
      )
    );
  });

  const weatherData = await Promise.all(requests);
  weatherData.forEach((response, index) => {
    savedCities.value[index].weather = response.data;
  });
};
await getCities()

const router = useRouter()
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: { state: city.state, city: city.city },
    query: {
      id: city.id,
      lat: city.coord.lat,
      lng: city.coord.lng,
    },
  });
};
</script>
