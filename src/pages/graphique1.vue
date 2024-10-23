<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import * as Plot from '@observablehq/plot'
import PlotFigure from '@/components/PlotFigure.js'
// import data25 from '@/assets/data25.json'
//import data29 from '@/assets/data29.json'
import { computed, ref, watch } from 'vue'

const data29 = await fetch('data29.json').then((res) => res.json())
const data25 = await fetch('data25.json').then((res) => res.json())

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
  <div class="flex flex-col items-center justify-center">
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
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center mt-20 mb-6">Graphique</h1>
    <p class="mb-4">
      Le graphique présente une comparaison des températures maximales mensuelles entre deux
      départements français, le Département 25 (représenté en rouge) et le Département 29
      (représenté en bleu), pour une année donnée. Chaque point sur le graphique représente la
      température maximale enregistrée pour un mois donné dans les deux départements, avec l'axe des
      abscisses (X) indiquant les mois (de 1 à 12) et l'axe des ordonnées (Y) représentant les
      températures maximales en degrés Celsius.
    </p>
    <h1 class="text-4xl font-bold text-center mb-6">Analyse</h1>
    <p class="mb-4">
      Les valeurs observées : Le Département 25 (en bleu) et le Département 29 (en orange) affichent
      une tendance similaire, avec des températures maximales croissantes durant les mois d'été (mai
      à août), puis décroissantes vers l'automne et l'hiver. En juin, juillet, et août, les
      températures maximales atteignent des sommets pour les deux départements. Cependant, il existe
      des variations légères, notamment pour certains mois comme avril et octobre, où le Département
      25 enregistre des températures légèrement plus élevées que le Département 29, et inversement
      pour d'autres mois. Répartition des températures : Les variations observées indiquent que les
      deux départements, bien qu'éloignés géographiquement, connaissent des températures maximales
      similaires durant l'année. Cela suggère des conditions climatiques relativement comparables,
      mais avec des nuances potentielles dans des mois spécifiques. Les écarts peuvent être
      attribués à des facteurs locaux tels que l'altitude, la proximité avec la mer (surtout pour le
      Département 29), ou d'autres caractéristiques géographiques.
    </p>
    <p class="mb-4">
      Dans le cadre du projet, qui vise à analyser les changements climatiques en France à
      différentes échelles, cette visualisation peut illustrer des différences locales importantes
      dans l'évolution des températures maximales entre 1964 et 2021. En comparant les tendances sur
      cette période, il est possible de mettre en lumière des augmentations significatives des
      températures dans certains départements, reflétant l'impact du réchauffement climatique. Ce
      type d'analyse permet aux citoyens et aux décideurs de mieux comprendre comment certaines
      régions sont plus affectées que d'autres par l'augmentation des températures, et souligne
      l'importance de stratégies d'adaptation et d'atténuation à l'échelle locale. Ces informations
      sont cruciales pour anticiper les effets potentiels du changement climatique sur les
      écosystèmes, l'agriculture et les infrastructures.
    </p>
    <p class="text-sm text-gray-500">Source</p>
    <p class="text-sm text-blue-500 hover:underline transition duration-300">
      <a href="https://defis.data.gouv.fr/datasets/6569b3d7d193b4daf2b43edc" target="_blank">
        https://defis.data.gouv.fr/datasets/6569b3d7d193b4daf2b43edc
      </a>
    </p>
  </div>
</template>

<style scoped>
.hover-info {
  font-size: 16px;
  font-weight: bold;
  color: #333;
}
</style>
