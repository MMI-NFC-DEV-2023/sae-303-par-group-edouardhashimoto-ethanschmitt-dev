<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import LoginOut from './components/LoginOut.vue';
</script>

<template>
  <header>
    <div class="wrapper">
      <h1 class="text-2xl bg-slate-400">Template Principal (Menu/Footer/...)</h1>
      <LoginOut />
      <nav>
        <ul>
          <RouterLink to="/">Accueil</RouterLink>
        </ul>
      </nav>
    </div>
  </header>
  <main>
    <RouterView v-slot="{ Component }">
      <Suspense>
        <component :is="Component" :key="$route.path" />
      </Suspense>
    </RouterView>
  </main>
</template>
