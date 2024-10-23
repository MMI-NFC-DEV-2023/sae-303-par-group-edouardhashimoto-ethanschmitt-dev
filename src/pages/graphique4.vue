<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import * as Plot from '@observablehq/plot'
import PlotFigure from '@/components/PlotFigure.js'
import departement25Doubs from '@/assets/departement-25-doubsGeojson.json'
// import data25 from '@/assets/data25.json'
import { ref, computed, watch } from 'vue'

// Chargement des données
const data25 = await fetch('data25.json').then((res) => res.json())

const monthIndexArray = ['Oct', 'Nov', 'Dec', 'Jan', 'Feb', 'Mar', 'Apr']

const selectedYearNeige = ref(Math.min(...data25.map((d) => d.AAAAMM.slice(0, 4))))
const selectedMonthNeige = ref(monthIndexArray[0])

// Transformation des données
const neigeData25 = computed(() =>
  data25.map((d) => {
    const year = d.AAAAMM.slice(0, 4)
    const month = d.AAAAMM.slice(4, 6) - 1
    return {
      date: new Date(year, month),
      NBJNEIGETOT1: +d.NBJNEIGETOT1 || 0,
      NBJNEIGETOT10: +d.NBJNEIGETOT10 || 0,
      NBJNEIGETOT30: +d.NBJNEIGETOT30 || 0,
      longitude: +d.LON,
      latitude: +d.LAT
    }
  })
)

// Filtrage des données selon l'année et le mois sélectionnés
const filteredNeigeData = computed(() => {
  return neigeData25.value.filter(
    (d) =>
      d.date.getFullYear() === +selectedYearNeige.value &&
      d.date.getMonth() === monthIndexArray.indexOf(selectedMonthNeige.value) &&
      d.NBJNEIGETOT1 !== 0
  )
})

// Options de graphique
const plotOptionsNeige = computed(() => ({
  projection: {
    type: 'mercator',
    domain: departement25Doubs
  },
  marks: [
    Plot.geo(departement25Doubs, {
      fill: '#b3e5fc',
      stroke: '#666'
    }),
    ...(filteredNeigeData.value.length > 0
      ? [
          Plot.dot(filteredNeigeData.value, {
            x: 'longitude',
            y: 'latitude',
            fill: (d) =>
              d.NBJNEIGETOT1 > 30 ? '#001f4d' : d.NBJNEIGETOT1 > 10 ? '#4d79ff' : '#b3c6ff',
            r: (d) => Math.sqrt(d.NBJNEIGETOT1) * 3,
            stroke: '#ffffff',
            strokeWidth: 2.5
          }),
          Plot.text(filteredNeigeData.value, {
            x: 'longitude',
            y: 'latitude',
            text: (d) => `${d.NBJNEIGETOT1} jours`,
            dx: 20,
            dy: -20,
            fontSize: 16,
            fontWeight: 'bold',
            fill: '#001f4d',
            stroke: 'white',
            strokeWidth: 0.5
          })
        ]
      : [])
  ],
  width: 1000,
  height: 600,
  margin: 20,
  animation: true
}))

const plotKey = ref(0)

// Watch pour les changements de l'année et du mois sélectionnés
watch([selectedYearNeige, selectedMonthNeige], () => {
  plotKey.value += 1 // Incrémente pour forcer la mise à jour
})
</script>

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center mt-20 mb-6">
      Visualisation de l'enneigement dans le Doubs
    </h1>

    <div class="flex flex-col items-center">
      <label for="year-select" class="mb-2">Sélectionnez une année : {{ selectedYearNeige }}</label>
      <input
        type="range"
        v-model="selectedYearNeige"
        :min="Math.min(...data25.map((d) => d.AAAAMM.slice(0, 4)))"
        :max="Math.max(...data25.map((d) => d.AAAAMM.slice(0, 4)))"
        step="1"
        class="mb-4"
      />

      <label for="month-select" class="mb-2">Sélectionnez un mois :</label>
      <select v-model="selectedMonthNeige" id="month-select" class="mb-4">
        <option v-for="month in monthIndexArray" :key="month" :value="month">{{ month }}</option>
      </select>
    </div>

    <div class="flex justify-center">
      <PlotFigure :key="plotKey" :options="plotOptionsNeige" />
    </div>
  </div>
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center mt-20 mb-6">Graphique</h1>
    <p class="mb-4">
      Le graphique présente une visualisation des jours d'enneigement dans le Département 25 (Doubs)
      pour des périodes spécifiques, en comparant le nombre de jours avec plus de 1 cm, 10 cm, et 30
      cm de neige. La carte géographique illustre également la répartition géographique des jours
      d'enneigement à l'aide de points codés par couleur et taille.
    </p>
    <h1 class="text-4xl font-bold text-center mb-6">Analyse</h1>
    <p class="mb-4">
      <strong>Les valeurs observées :</strong> Le graphique montre le nombre total de jours
      d'enneigement pour le Département 25. Par exemple, il est possible d'observer un nombre
      significatif de jours avec plus de 1 cm de neige, représentant une base de l'analyse. Les
      jours avec plus de 10 cm et 30 cm de neige sont également présentés, permettant de visualiser
      la variabilité de l'enneigement.
    </p>
    <p class="mb-4">
      <strong>Visualisation :</strong> La carte utilise une palette de couleurs dégradées allant du
      bleu clair au bleu foncé, indiquant des niveaux d'enneigement différents. Les points
      représentant les jours d'enneigement sont agrandis pour plus de visibilité, et chaque point
      est entouré d'un contour blanc pour un meilleur contraste. Les annotations sur la carte
      fournissent une compréhension immédiate des jours d'enneigement, avec des chiffres affichés
      au-dessus des points, ce qui facilite l'interprétation des données.
    </p>
    <p class="mb-4">
      Dans le cadre du projet visant à analyser les changements climatiques en France, cette
      visualisation des jours d'enneigement permet aux citoyens et aux décideurs de mieux comprendre
      les dynamiques locales liées à l'enneigement. Cela aide à contextualiser comment le changement
      climatique pourrait affecter la fréquence et l'intensité des épisodes neigeux, des
      informations essentielles pour planifier les stratégies d'adaptation aux changements
      climatiques à l'échelle régionale.
    </p>
    <p class="text-sm text-gray-500">Source</p>
    <p class="text-sm text-blue-500 hover:underline transition duration-300">
      <a href="https://defis.data.gouv.fr/datasets/6569b3d7d193b4daf2b43edc" target="_blank">
        https://defis.data.gouv.fr/datasets/6569b3d7d193b4daf2b43edc
      </a>
    </p>
    <div class="flex justify-end mt-5">
      <router-link to="/graphique1">
        <button
          class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-700 transition duration-300"
        >
          Aller à Graphique 1
        </button>
      </router-link>
    </div>
  </div>
</template>

<style scoped>
.plot-container {
  display: flex;
  justify-content: space-between;
}
</style>
