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

function addToCart(lesson) {
  if (lesson.spaces > 0) {
    cart.value.push(lesson)
    lesson.spaces--
  }
}

const cartCount = computed(() => cart.value.length)
</script>

<template>
  <div class="app">
    <header class="app-header">
      <h1>After School Classes</h1>
      <button class="cart-toggle" @click="showCart = !showCart">
        Cart ({{ cartCount }})
      </button>
    </header>

    <main>
      <section v-if="!showCart" class="lessons-section">
        <h2>Available Lessons</h2>
        <div class="lessons-grid">
          <article v-for="lesson in lessons" :key="lesson.id" class="lesson-card">
            <div class="lesson-icon">{{ lesson.icon }}</div>
            <h3>{{ lesson.subject }}</h3>
            <p>{{ lesson.location }}</p>
            <p>£{{ lesson.price }}</p>
            <p>Spaces: {{ lesson.spaces }}</p>
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
        <div v-if="cart.length === 0">
          <p>Your cart is empty.</p>
        </div>
        <ul v-else>
          <li v-for="(item, index) in cart" :key="index">
            {{ item.subject }} - £{{ item.price }}
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
  font-family: system-ui, sans-serif;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.cart-toggle {
  padding: 0.5rem 1rem;
  border-radius: 999px;
  border: none;
  cursor: pointer;
}

.lessons-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
}

.lesson-card {
  padding: 1.5rem;
  border-radius: 1rem;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.lesson-icon {
  font-size: 2rem;
}

.add-button {
  margin-top: 0.75rem;
  padding: 0.5rem 1rem;
  border-radius: 999px;
  border: none;
  cursor: pointer;
}
</style>
