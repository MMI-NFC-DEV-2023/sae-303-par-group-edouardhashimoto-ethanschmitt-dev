<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import * as Plot from '@observablehq/plot'
import PlotFigure from '@/components/PlotFigure.js'
import data25 from '@/assets/data25.json'
import data29 from '@/assets/data29.json'
import { computed, ref, watch } from 'vue'

// Liste des années disponibles, extraite des données
const years = Array.from(new Set(data25.map((d) => d.AAAAMM.slice(0, 4))))

// Année sélectionnée (par défaut "1964")
const selectedYear = ref('1964') 

// Données du point survolé
const hoveredData = ref(null)

// Filtrer et transformer les données du département 25
const filteredData25 = computed(() => {
  const groupedData = data25
    .filter((d) => d.AAAAMM.slice(0, 4) === selectedYear.value) // Filtrer par année
    .map((d) => ({
      ...d,
      TX: +d.TX, // Convertir TX en nombre
      Mois: +d.AAAAMM.slice(4, 6) // Extraire le mois et convertir en nombre
    }))
    // Grouper par mois et garder uniquement la température maximale de chaque mois
    .reduce((acc, current) => {
      const mois = current.Mois
      if (!acc[mois] || current.TX > acc[mois].TX) {
        acc[mois] = current // Garder l'entrée avec la température maximale
      }
      return acc
    }, {})

  return Object.values(groupedData) // Retourner les valeurs sous forme de tableau
})

// Filtrer et transformer les données du département 29
const filteredData29 = computed(() => {
  const groupedData = data29
    .filter((d) => d.AAAAMM.slice(0, 4) === selectedYear.value) // Filtrer par année
    .map((d) => ({
      ...d,
      TX: +d.TX, // Convertir TX en nombre
      Mois: +d.AAAAMM.slice(4, 6) // Extraire le mois et convertir en nombre
    }))
    // Grouper par mois et garder uniquement la température maximale de chaque mois
    .reduce((acc, current) => {
      const mois = current.Mois
      if (!acc[mois] || current.TX > acc[mois].TX) {
        acc[mois] = current // Garder l'entrée avec la température maximale
      }
      return acc
    }, {})

  return Object.values(groupedData) // Retourner les valeurs sous forme de tableau
})

// Créer les données de graphique pour les deux départements
const plotData = computed(() => [
  ...filteredData25.value.map((d) => ({ département: '25', Mois: d.Mois, TX: d.TX })),
  ...filteredData29.value.map((d) => ({ département: '29', Mois: d.Mois, TX: d.TX }))
])

// Fonction pour mettre à jour les données survolées
const handleHover = (event, d) => {
  hoveredData.value = d ? `Département: ${d.département}, Mois: ${d.Mois}, TX: ${d.TX}°C` : null
}

// Options pour Plot
const plotOptions = computed(() => ({
  marks: [
    Plot.dot(plotData.value, {
      x: 'Mois',
      y: 'TX',
      stroke: 'département',
      fill: (d) => (d.département === '25' ? 'steelblue' : 'orange'),
      r: 5, // Taille du point
      title: (d) => `Département ${d.département}, Mois: ${d.Mois}, Température Max: ${d.TX}°C`,
      // Ajout de l'événement hover
      on: {
        pointerenter: (event, d) => handleHover(event, d),
        pointerleave: () => handleHover(null, null)
      }
    })
  ],
  y: { label: 'Température Max en °C' },
  x: { label: 'Mois', tickFormat: (d) => `${d}` },
  color: { legend: true }
}))

// Clé unique pour forcer la mise à jour du graphique
const plotKey = ref(0)

// Watch for changes in selectedYear to trigger reactivity
watch(selectedYear, () => {
  plotKey.value += 1 // Incrémente la clé pour forcer la mise à jour
})
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen">
    <h1>Température maximale par mois</h1>

    <label for="year-select">Choisir l'année :</label>
    <select v-model="selectedYear" id="year-select">
      <option v-for="year in years" :key="year" :value="year">{{ year }}</option>
    </select>

    <PlotFigure :key="plotKey" :options="plotOptions" />

    <!-- Afficher les données survolées -->
    <div v-if="hoveredData" class="hover-info">
      {{ hoveredData }}
    </div>
  </div>
</template>

<style scoped>
.hover-info {
  margin-top: 20px;
  font-size: 16px;
  font-weight: bold;
  color: #333;
}
</style>
