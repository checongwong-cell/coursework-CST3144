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

    <div class="weather-heading">
      <div>
        <h1>Weather App</h1>
        <p>Check the weather and forecast for your city.</p>
      </div>
      <span class="weather-heading-icon">☀️</span>
    </div>

    <form class="weather-search" @submit.prevent="fetchWeather">
      <input
        v-model="city"
        placeholder="Enter a city"
        aria-label="City name"
      />
      <button type="submit" :disabled="loading">
        {{ loading ? 'Loading...' : 'Get Weather' }}
      </button>
    </form>

    <p v-if="loading" class="weather-message">
      Loading weather...
    </p>

    <p v-if="error" class="weather-error">
      {{ error }}
    </p>

    <section v-if="weather" class="weather-result">

      <div class="weather-city">
        <h2>📍 {{ city }}</h2>
        <p>Current weather conditions</p>
      </div>

      <div class="weather-details">

        <div class="weather-detail-card">
          <span class="detail-icon">🌡️</span>
          <div>
            <p>Temperature</p>
            <h3>{{ weather.temperature }}</h3>
          </div>
        </div>

        <div class="weather-detail-card">
          <span class="detail-icon">💨</span>
          <div>
            <p>Wind</p>
            <h3>{{ weather.wind }}</h3>
          </div>
        </div>

        <div class="weather-detail-card">
          <span class="detail-icon">🌤️</span>
          <div>
            <p>Conditions</p>
            <h3>{{ weather.description }}</h3>
          </div>
        </div>

      </div>

      <div class="forecast-section">
        <h2>📅 Weather Forecast</h2>

        <div class="forecast-list">
          <div
            v-for="(day, index) in weather.forecast"
            :key="index"
            class="forecast-card"
          >
            <h3>{{ day.day }}</h3>
            <span class="forecast-icon">🌤️</span>
            <p class="forecast-temperature">
              {{ day.temperature }}
            </p>
            <p>💨 {{ day.wind }}</p>
          </div>
        </div>
      </div>

    </section>

  </main>
</template>
