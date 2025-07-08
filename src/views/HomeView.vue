<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city"
        class="py-2 px-1 w-full bg-transparent border-b-2 focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="searchResults"
      >
      <p v-if="searchError"> Sorry. something went wrong, please try again</p>
      <p v-if="!serverError && searchResults.length === 0"> No results found</p>
      <template v-else>
        <li
          v-for="searchResult in searchResults"
          :key="searchResult.id"
          class="py-2 cursor-pointer"
          @click="previewCity(searchResult)"
        >
          {{ searchResult.place_name }}
        </li>
      </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList/>
        <template #fallbback>
          <p>Loading...</p>
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRoute } from 'vue-router'

const searchQuery = ref('')
const queryTimeout = ref(null)
const searchResults = ref(null)
const searchError = ref(null)
const router = useRoute()

const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(', ')
  router.push ({
    name: 'cityView',
    params: {
      city,
      state,
    },
    query: {
      lat: searchResult.center[1],
      lng: searchResult.center[0],
      preview: true,
    },
  })
}

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
        const response = await axios.get(
          `https://api.mapbox.org/geocoding/v5/mapbox.places/${searchQuery.value}.json`
        )
        searchResults.value = response.data.features
      } catch  {
        searchError.value = true
      }
      return
    }
    searchResults.value = null
  }, 300)
}
</script>
