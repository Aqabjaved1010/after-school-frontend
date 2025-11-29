<script setup>
import { ref, computed, onMounted } from 'vue'

const showCart = ref(false)

const lessons = ref([])

async function loadLessons() {
  try {
    const res = await fetch('https://after-school-backend-r4mc.onrender.com/lessons')
    const data = await res.json()
    lessons.value = data.map(l => ({
      ...l,
      id: l._id
    }))
  } catch (err) {
    console.error('Failed to load lessons', err)
  }
}


onMounted(() => {
  loadLessons()
})



const cart = ref([])

const sortBy = ref('subject')
const sortOrder = ref('asc')

const searchTerm = ref('')

const customerName = ref('')
const customerPhone = ref('')
const checkoutMessage = ref('')

function addToCart(lesson) {
  if (lesson.spaces > 0) {
    cart.value.push(lesson)
    lesson.spaces--
  }
}

function removeFromCart(index) {
  const item = cart.value[index]
  if (item) {
    const lesson = lessons.value.find(l => l.id === item.id)
    if (lesson) {
      lesson.spaces++
    }
    cart.value.splice(index, 1)
  }
}

const cartCount = computed(() => cart.value.length)

const filteredLessons = computed(() => {
  const term = searchTerm.value.trim().toLowerCase()
  if (!term) return lessons.value
  return lessons.value.filter(lesson => {
    const subject = lesson.subject.toLowerCase()
    const location = lesson.location.toLowerCase()
    const price = String(lesson.price).toLowerCase()
    const spaces = String(lesson.spaces).toLowerCase()
    return (
      subject.includes(term) ||
      location.includes(term) ||
      price.includes(term) ||
      spaces.includes(term)
    )
  })
})

const sortedLessons = computed(() => {
  const list = [...filteredLessons.value]
  list.sort((a, b) => {
    let result = 0
    if (sortBy.value === 'subject' || sortBy.value === 'location') {
      result = a[sortBy.value].localeCompare(b[sortBy.value])
    } else if (sortBy.value === 'price' || sortBy.value === 'spaces') {
      if (a[sortBy.value] < b[sortBy.value]) result = -1
      else if (a[sortBy.value] > b[sortBy.value]) result = 1
      else result = 0
    }
    return sortOrder.value === 'asc' ? result : -result
  })
  return list
})


const isNameValid = computed(() => {
  const value = customerName.value.trim()
  if (value.length === 0) return false
  return /^[A-Za-z\s]+$/.test(value)
})

const isPhoneValid = computed(() => {
  const value = customerPhone.value.trim()
  if (value.length === 0) return false
  return /^[0-9]+$/.test(value)
})

const canCheckout = computed(() => {
  return cart.value.length > 0 && isNameValid.value && isPhoneValid.value
})

async function checkout() {
  if (!canCheckout.value) return

  const name = customerName.value.trim()
  const phone = customerPhone.value.trim()

  const lessonCounts = {}
  for (const item of cart.value) {
    const id = item.id
    if (!lessonCounts[id]) {
      lessonCounts[id] = 0
    }
    lessonCounts[id]++
  }

  const items = Object.entries(lessonCounts).map(([lessonId, spaces]) => ({
    lessonId,
    spaces
  }))

  const order = {
    customerName: name,
    customerPhone: phone,
    items
  }

  try {
    const res = await fetch('http://localhost:4000/orders', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(order)
    })

    if (!res.ok) {
      checkoutMessage.value = 'Failed to submit order.'
      return
    }

    for (const item of items) {
      await fetch(`http://localhost:4000/lessons/${item.lessonId}/spaces`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ change: -item.spaces })
      })
    }

    checkoutMessage.value = 'Order submitted successfully for ' + name + '.'
    customerName.value = ''
    customerPhone.value = ''
    cart.value = []
  } catch (error) {
    checkoutMessage.value = 'Failed to submit order.'
  }
}

</script>

<template>
  <div class="app">
    <header class="app-header">
      <h1>After School Classes</h1>
        <button
          class="cart-toggle"
          @click="showCart = !showCart"
          :disabled="!showCart && cartCount === 0"
>
           {{ showCart ? 'Back to Lessons' : 'Cart (' + cartCount + ')' }}
        </button>

    </header>

    <main>
      <section v-if="!showCart" class="lessons-section">
        <div class="controls-bar">
          <h2>Available Lessons</h2>
          <div class="controls-right">
            <input
              class="search-input"
              type="text"
              v-model="searchTerm"
              placeholder="Search lessons..."
            />
            <div class="sort-controls">
              <label class="sort-control">
                <span>Sort by</span>
                <select v-model="sortBy">
                  <option value="subject">Subject</option>
                  <option value="location">Location</option>
                  <option value="price">Price</option>
                  <option value="spaces">Spaces</option>
                </select>
              </label>
              <label class="sort-control">
                <span>Order</span>
                <select v-model="sortOrder">
                  <option value="asc">Ascending</option>
                  <option value="desc">Descending</option>
                </select>
              </label>
            </div>
          </div>
        </div>


        <div class="lessons-grid">
          <article
            v-for="lesson in sortedLessons"
            :key="lesson.id"
            class="lesson-card"
          >
            <div class="lesson-icon">{{ lesson.icon }}</div>
            <h3>{{ lesson.subject }}</h3>
            <p class="lesson-location">{{ lesson.location }}</p>
            <p class="lesson-price">£{{ lesson.price }}</p>
            <p class="lesson-spaces">Spaces: {{ lesson.spaces }}</p>
            <button
              class="add-button"
              :disabled="lesson.spaces === 0"
              @click="addToCart(lesson)"
            >
              Add to Cart
            </button>
          </article>
        </div>
      </section>

      <section v-else class="cart-section">
        <h2>Shopping Cart</h2>
        <div v-if="cart.length === 0" class="cart-empty">
          <p>Your cart is empty.</p>
        </div>
        <ul v-else class="cart-list">
          <li v-for="(item, index) in cart" :key="index" class="cart-item">
            <div class="cart-item-info">
              <span class="cart-item-title">{{ item.subject }}</span>
              <span class="cart-item-location">{{ item.location }}</span>
            </div>
            <div class="cart-item-right">
              <span class="cart-item-price">£{{ item.price }}</span>
              <button class="remove-button" @click="removeFromCart(index)">
                Remove
              </button>
            </div>
          </li>
        </ul>

        <div class="checkout-area">
          <h3>Checkout</h3>
          <form class="checkout-form" @submit.prevent="checkout">
            <div class="form-row">
              <label>
                Name
                <input
                  type="text"
                  v-model="customerName"
                  placeholder="Enter your name"
                />
              </label>
              <p v-if="customerName && !isNameValid" class="error-text">
                Name must contain letters and spaces only.
              </p>
            </div>

            <div class="form-row">
              <label>
                Phone
                <input
                  type="text"
                  v-model="customerPhone"
                  placeholder="Enter your phone number"
                />
              </label>
              <p v-if="customerPhone && !isPhoneValid" class="error-text">
                Phone must contain numbers only.
              </p>
            </div>

            <button
              type="submit"
              class="checkout-button"
              :disabled="!canCheckout"
            >
              Checkout
            </button>
          </form>

          <p v-if="checkoutMessage" class="success-text">
            {{ checkoutMessage }}
          </p>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1rem;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.cart-toggle {
  padding: 0.6rem 1.2rem;
  border-radius: 999px;
  border: none;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s ease;
}

.cart-toggle:hover:not(:disabled) {
  background: black;
  color: white;
}

.cart-toggle:disabled {
  background: #ddd;
  color: #777;
  cursor: not-allowed;
}

.controls-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.8rem;
  gap: 1rem;
}


.sort-controls {
  display: flex;
  align-items: center;
  gap: 0.6rem;
}

.sort-control {
  display: flex;
  flex-direction: column;
  font-size: 0.75rem;
}

.sort-control select {
  height: 36px;
  padding: 0 0.6rem;
  border-radius: 999px;
  border: 1px solid #ccc;
  font-size: 0.8rem;
}


.controls-right {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}


.search-input {
  height: 36px;
  padding: 0 0.75rem;
  border-radius: 999px;
  border: 1px solid #ccc;
  min-width: 220px;
  font-size: 0.85rem;
}



.lessons-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
}

.lesson-card {
  padding: 1.5rem;
  border-radius: 1rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  transition: transform 0.2s ease;
}

.lesson-card:hover {
  transform: translateY(-4px);
}

.lesson-icon {
  font-size: 2rem;
}

.lesson-location {
  opacity: 0.8;
}

.lesson-price {
  font-weight: 600;
}

.lesson-spaces {
  margin-top: 0.25rem;
}

.add-button {
  margin-top: 0.75rem;
  padding: 0.6rem 1.2rem;
  border-radius: 999px;
  border: 1px solid black;
  background: white;
  color: black;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s ease;
}

.add-button:hover {
  background: black;
  color: white;
}

.add-button:disabled {
  background: #ddd;
  color: #777;
  border: none;
  cursor: not-allowed;
}

.cart-section {
  max-width: 700px;
  margin: 0 auto;
}

.cart-list {
  list-style: none;
  padding: 0;
  margin: 1rem 0 0;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.cart-item {
  padding: 0.75rem 1rem;
  border-radius: 0.75rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.cart-item-info {
  display: flex;
  flex-direction: column;
}

.cart-item-title {
  font-weight: 600;
}

.cart-item-location {
  font-size: 0.85rem;
  opacity: 0.8;
}

.cart-item-right {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.cart-item-price {
  font-weight: 600;
}

.remove-button {
  padding: 0.4rem 0.9rem;
  border-radius: 999px;
  border: 1px solid black;
  background: white;
  cursor: pointer;
  font-size: 0.85rem;
  transition: all 0.2s ease;
}

.remove-button:hover {
  background: black;
  color: white;
}

.checkout-area {
  margin-top: 2rem;
  padding-top: 1.5rem;
  border-top: 1px solid #eee;
}

.checkout-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-top: 0.5rem;
}

.form-row {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.form-row label {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  font-size: 0.9rem;
}

.form-row input {
  padding: 0.5rem 0.75rem;
  border-radius: 0.5rem;
  border: 1px solid #ccc;
}

.checkout-button {
  align-self: flex-start;
  padding: 0.6rem 1.4rem;
  border-radius: 999px;
  border: none;
  cursor: pointer;
  font-weight: 600;
  background: black;
  color: white;
  transition: all 0.2s ease;
}

.checkout-button:disabled {
  background: #ddd;
  color: #777;
  cursor: not-allowed;
}

.error-text {
  font-size: 0.8rem;
  color: #b00020;
}

.success-text {
  margin-top: 1rem;
  font-size: 0.9rem;
  color: #0a7a28;
}
</style>
