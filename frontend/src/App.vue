
<script setup>
import { ref, computed, onMounted } from 'vue'
import WeatherApp from './weatherapp.vue'

const API_URL = 'https://m01083489-server.onrender.com'
const LOCAL_API_URL = 'http://localhost:3000'

const showCart = ref(false)

const cart = ref([])

const customer = ref({
  name: '',
  phone: '',
  address: ''
})

const lessons = ref([])

// Search variables
const searchQuery = ref('')
const searchResults = ref(null)
const searchError = ref('')
const isSearching = ref(false)

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + item.price, 0)
})

// Remove lesson from cart and restore its available space
const removeFromCart = (index) => {
  const lesson = cart.value[index]

  const originalLesson = lessons.value.find(
    item => item.id === lesson.id
  )

  if (originalLesson) {
    originalLesson.space++
  }

  cart.value.splice(index, 1)
}

// Add lesson to cart
const addToCart = (lesson) => {
  if (lesson.space > 0) {
    lesson.space--
    cart.value.push({ ...lesson })
  }
}

// Fetch lessons from the backend
const fetchLessons = async () => {
  try {
    const response = await fetch(`${API_URL}/lessons`)

    if (!response.ok) {
      throw new Error('Failed to fetch lessons')
    }

    const data = await response.json()
    lessons.value = data

  } catch (error) {
    console.error('Error fetching lessons:', error)
  }
}

// Search lessons using the backend API
const searchLessons = async () => {
  if (searchQuery.value.trim() === '') {
    clearSearch()
    return
  }

  try {
    isSearching.value = true
    searchError.value = ''

    const response = await fetch(
  `${API_URL}/search?q=${encodeURIComponent(searchQuery.value)}`
)

    if (!response.ok) {
      throw new Error('Search failed')
    }

    const data = await response.json()
    searchResults.value = data

  } catch (error) {
    console.error('Search error:', error)
    searchError.value = 'Unable to search lessons. Please try again.'
  } finally {
    isSearching.value = false
  }
}

// Clear search and show all lessons
const clearSearch = () => {
  searchQuery.value = ''
  searchResults.value = null
  searchError.value = ''
}

// Checkout and submit order
const checkout = async () => {
  if (
    customer.value.name.trim() === '' ||
    customer.value.phone.trim() === '' ||
    customer.value.address.trim() === ''
  ) {
    alert('Please fill in all fields.')
    return
  }

  if (cart.value.length === 0) {
    alert('Your cart is empty.')
    return
  }

  try {
    // Save order to MongoDB
    const response = await fetch(`${API_URL}/orders`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        customer: customer.value,
        items: cart.value,
        total: cartTotal.value
      })
    })

    if (!response.ok) {
      throw new Error('Order submission failed')
    }

    // Find unique lesson IDs in the cart
    const updatedLessonIds = [
      ...new Set(cart.value.map(item => item.id))
    ]

    // Update available spaces in MongoDB
    for (const lessonId of updatedLessonIds) {
      const lesson = lessons.value.find(
        item => item.id === lessonId
      )

      if (!lesson) {
        throw new Error('Lesson not found')
      }

      const updateResponse = await fetch(
        `${API_URL}/lessons/${lessonId}`,
        {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            space: lesson.space
          })
        }
      )

      if (!updateResponse.ok) {
        throw new Error('Failed to update lesson spaces')
      }
    }

    // Refresh lessons using updated database information
    await fetchLessons()

    alert(`Thank you ${customer.value.name}! Your order has been placed.`)

    // Clear cart and customer details
    cart.value = []

    customer.value = {
      name: '',
      phone: '',
      address: ''
    }

    showCart.value = false

  } catch (error) {
    console.error('Checkout error:', error)
    alert('Unable to place your order. Please try again.')
  }
}

// Load lessons when the application starts
onMounted(() => {
  fetchLessons()
})

// Sorting options
const sortBy = ref('topic')
const sortOrder = ref('ascending')

// Sort all lessons or search results
const sortedLessons = computed(() => {
  const sourceLessons = searchResults.value === null
    ? lessons.value
    : searchResults.value

  const sorted = [...sourceLessons]

  sorted.sort((a, b) => {
    let valueA = a[sortBy.value]
    let valueB = b[sortBy.value]

    if (typeof valueA === 'string') {
      valueA = valueA.toLowerCase()
      valueB = valueB.toLowerCase()
    }

    if (valueA < valueB) {
      return sortOrder.value === 'ascending' ? -1 : 1
    }

    if (valueA > valueB) {
      return sortOrder.value === 'ascending' ? 1 : -1
    }

    return 0
  })

  return sorted
})
</script>

<template>
  <div class="app">

    <header>
      <h1>After School Activities</h1>
      <p>Find and book your favourite lessons.</p>

      <button class="cart-button" @click="showCart = !showCart">
        🛒 Cart: {{ cart.length }}
      </button>
    </header>

    <!-- CART AND CHECKOUT PAGE -->
    <main v-if="showCart">

      <button @click="showCart = false">
        ← Back to Lessons
      </button>

      <div class="cart-items">
        <h2>Your Cart</h2>

        <p v-if="cart.length === 0">
          Your cart is empty.
        </p>

        <div
          v-for="(item, index) in cart"
          :key="index"
          class="cart-item"
        >
          <span>{{ item.topic }}</span>
          <span>£{{ item.price }}</span>

          <button @click="removeFromCart(index)">
            Remove
          </button>
        </div>

        <p>
          <strong>Total: £{{ cartTotal }}</strong>
        </p>
      </div>

      <div v-if="cart.length > 0" class="checkout">
        <h2>Checkout</h2>

        <label>
          Name:
          <input v-model="customer.name" type="text">
        </label>

        <label>
          Phone:
          <input v-model="customer.phone" type="tel">
        </label>

        <label>
          Address:
          <input v-model="customer.address" type="text">
        </label>

        <button @click="checkout">
          Checkout
        </button>
      </div>

    </main>

    <!-- LESSONS PAGE -->
    <main v-else>

      <WeatherApp />

      <h2>Lessons</h2>

      <!-- SEARCH SECTION -->
      <div class="search-section">

        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search by subject, location, price or spaces..."
          @keyup.enter="searchLessons"
        >

        <button
          @click="searchLessons"
          :disabled="isSearching"
        >
          {{ isSearching ? 'Searching...' : 'Search' }}
        </button>

        <button
          v-if="searchResults !== null"
          @click="clearSearch"
        >
          Clear
        </button>

      </div>

      <p v-if="searchError">
        {{ searchError }}
      </p>

      <p v-else-if="searchResults !== null && searchResults.length === 0">
        No matching lessons found.
      </p>

      <p v-else-if="searchResults !== null">
        Found {{ searchResults.length }} matching lessons.
      </p>

      <!-- SORTING -->
      <div class="sorting">

        <label for="sortBy">Sort by:</label>

        <select id="sortBy" v-model="sortBy">
          <option value="topic">Subject</option>
          <option value="location">Location</option>
          <option value="price">Price</option>
          <option value="space">Spaces</option>
        </select>

        <label for="sortOrder">Order:</label>

        <select id="sortOrder" v-model="sortOrder">
          <option value="ascending">Ascending</option>
          <option value="descending">Descending</option>
        </select>

      </div>

      <!-- LESSON CARDS -->
      <div class="lessons">

        <div
          v-for="lesson in sortedLessons"
          :key="lesson.id"
          class="lesson-card"
        >

          <div class="lesson-icon">
            <img :src="lesson.image" :alt="lesson.topic">
          </div>

          <h3>{{ lesson.topic }}</h3>

          <p>
            <strong>Location:</strong>
            {{ lesson.location }}
          </p>

          <p>
            <strong>Price:</strong>
            £{{ lesson.price }}
          </p>

          <p>
            <strong>Spaces available:</strong>
            {{ lesson.space }}
          </p>

          <button
            @click="addToCart(lesson)"
            :disabled="lesson.space === 0"
          >
            {{ lesson.space === 0 ? 'Sold Out' : 'Add to Cart' }}
          </button>

        </div>

      </div>

    </main>

  </div>
</template>
