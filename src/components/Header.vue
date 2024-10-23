<script setup lang="ts">
import { RouterLink } from 'vue-router'
</script>

<template>
  <header class="bg-gray-800 text-white p-4 shadow-md">
    <div class="container mx-auto flex justify-between items-center">
      <h1 class="text-3xl font-bold">SAE303</h1>
      <nav>
        <ul class="flex space-x-4">
          <li>
            <RouterLink to="/graphique1" class="hover:text-gray-400 transition duration-300">Graph1</RouterLink>
          </li>
          <li>
            <RouterLink to="/graphique2" class="hover:text-gray-400 transition duration-300">Graph2</RouterLink>
          </li>
          <li>
            <RouterLink to="/graphique3" class="hover:text-gray-400 transition duration-300">Graph3</RouterLink>
          </li>
          <li>
            <RouterLink to="/equipes" class="hover:text-gray-400 transition duration-300">test</RouterLink>
          </li>
        </ul>
      </nav>
    </div>
  </header>
</template>

