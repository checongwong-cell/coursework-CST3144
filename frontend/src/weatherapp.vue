
<script setup>
import { ref, onMounted } from 'vue'

const city = ref('London')
const weather = ref(null)
const loading = ref(false)
const error = ref('')

async function fetchWeather() {
  loading.value = true
  error.value = ''

  try {
    const url =
      `https://goweather.xyz/v2/weather/${encodeURIComponent(city.value)}`

    const response = await fetch(url)

    if (!response.ok) {
      throw new Error('Failed to fetch weather')
    }

    weather.value = await response.json()
  } catch (err) {
    error.value = 'Unable to load weather. Please try again.'
    weather.value = null
  } finally {
    loading.value = false
  }
}

onMounted(fetchWeather)
</script>

<template>
  <main class="weather-app">
    <h1>Weather App</h1>

    <form @submit.prevent="fetchWeather">
      <input v-model="city" placeholder="Enter a city" />
      <button type="submit">Get Weather</button>
    </form>

    <p v-if="loading">Loading weather...</p>
    <p v-if="error">{{ error }}</p>

    <section v-if="weather" class="weather-result">
      <h2>{{ city }}</h2>
      <p>Temperature: {{ weather.temperature }}</p>
      <p>Wind: {{ weather.wind }}</p>
      <p>Conditions: {{ weather.description }}</p>

      <h3>Forecast</h3>

      <ul>
        <li
          v-for="(day, index) in weather.forecast"
          :key="index"
        >
          {{ day.day }} — {{ day.temperature }},
          Wind: {{ day.wind }}
        </li>
      </ul>
    </section>
  </main>
</template>
