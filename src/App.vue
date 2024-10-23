<script setup lang="ts">
import { RouterView } from 'vue-router'
import Header from './components/Header.vue'
import Footer from './components/footer.vue'
</script>

<template>
  <Header></Header>
  <main class="container mx-auto p-4">
    <RouterView v-slot="{ Component }">
      <Suspense>
        <component :is="Component" :key="$route.path" />
      </Suspense>
    </RouterView>
  </main>
  <Footer></Footer>
</template>