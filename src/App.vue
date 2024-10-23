<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <div class="wrapper">
      <h1 class="text-2xl bg-slate-400">SAE303</h1>
      <nav>
        <ul>
          <RouterLink to="/graphique1">Graph1</RouterLink>
        </ul>
        <ul>
          <RouterLink to="/graphique2">Graph2</RouterLink>
        </ul>
        <ul>
          <RouterLink to="/graphique3">Graph3</RouterLink>
        </ul>
        <ul>
          <RouterLink to="/equipes">test</RouterLink>
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
