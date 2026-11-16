<script setup>
import { ref, computed, onMounted } from 'vue'
const showCart = ref(false)

const cart = ref([])

const customer = ref({
  name: '',
  phone: '',
  address: ''
})

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + item.price, 0)
})

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

const addToCart = (lesson) => {
  if (lesson.space > 0) {
    lesson.space--
    cart.value.push(lesson)
  }
}

const checkout = async () => {
  if (
    customer.value.name === '' ||
    customer.value.phone === '' ||
    customer.value.address === ''
  ) {
    alert('Please fill in all fields.')
    return
  }

  try {
    const response = await fetch('https://m01083489-server.onrender.com/orders', {
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
    
    for (const item of cart.value) {
  const updateResponse = await fetch(
    `https://m01083489-server.onrender.com/lessons/${item.id}`,
    {
      method: 'PUT'
    }
  )

  if (!updateResponse.ok) {
    throw new Error('Failed to update lesson spaces')
  }
}

await fetchLessons()

    alert(`Thank you ${customer.value.name}! Your order has been placed.`)

    cart.value = []

    customer.value = {
      name: '',
      phone: '',
      address: ''
    }

  } catch (error) {
    console.error('Checkout error:', error)
    alert('Unable to place your order. Please try again.')
  }
}

  alert(`Thank you ${customer.value.name}! Your order has been placed.`)

  cart.value = []

  customer.value = {
    name: '',
    phone: '',
    address: ''
  }


const lessons = ref([])

const fetchLessons = async () => {
  try {
    const response = await fetch('https://m01083489-server.onrender.com/lessons')

    if (!response.ok) {
      throw new Error('Failed to fetch lessons')
    }

    const data = await response.json()
    lessons.value = data
  } catch (error) {
    console.error('Error fetching lessons:', error)
  }
}

onMounted(() => {
  fetchLessons()
})

const sortBy = ref('topic')
const sortOrder = ref('ascending')

const sortedLessons = computed(() => {
  const sorted = [...lessons.value]

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

```vue
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

      <h2>Lessons</h2>

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
            Add to Cart
          </button>

        </div>

      </div>

    </main>

  </div>
</template>
```