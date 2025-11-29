<script setup>
import { ref, computed } from 'vue'

const showCart = ref(false)

const lessons = ref([
  { id: 1, subject: 'Maths', location: 'Hendon', price: 100, spaces: 5, icon: '📘' },
  { id: 2, subject: 'English', location: 'Colindale', price: 90, spaces: 5, icon: '📗' },
  { id: 3, subject: 'Science', location: 'Brent Cross', price: 110, spaces: 5, icon: '🔬' },
  { id: 4, subject: 'History', location: 'Golders Green', price: 80, spaces: 5, icon: '📜' },
  { id: 5, subject: 'Coding', location: 'Hendon', price: 120, spaces: 5, icon: '💻' },
  { id: 6, subject: 'Art', location: 'Colindale', price: 70, spaces: 5, icon: '🎨' },
  { id: 7, subject: 'Music', location: 'Brent Cross', price: 95, spaces: 5, icon: '🎵' },
  { id: 8, subject: 'Drama', location: 'Golders Green', price: 85, spaces: 5, icon: '🎭' },
  { id: 9, subject: 'Robotics', location: 'Hendon', price: 130, spaces: 5, icon: '🤖' },
  { id: 10, subject: 'Sports', location: 'Colindale', price: 75, spaces: 5, icon: '🏅' }
])

const cart = ref([])

const sortBy = ref('subject')
const sortOrder = ref('asc')

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

const sortedLessons = computed(() => {
  const list = [...lessons.value]
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
</script>

<template>
  <div class="app">
    <header class="app-header">
      <h1>After School Classes</h1>
      <button
        class="cart-toggle"
        @click="showCart = !showCart"
        :disabled="cartCount === 0"
      >
        Cart ({{ cartCount }})
      </button>
    </header>

    <main>
      <section v-if="!showCart" class="lessons-section">
        <div class="controls-bar">
          <h2>Available Lessons</h2>
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
  margin-bottom: 1.5rem;
  gap: 1rem;
  flex-wrap: wrap;
}

.sort-controls {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.sort-control {
  display: flex;
  flex-direction: column;
  font-size: 0.9rem;
}

.sort-control select {
  margin-top: 0.25rem;
  padding: 0.3rem 0.5rem;
  border-radius: 0.5rem;
  border: 1px solid #ccc;
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
</style>
