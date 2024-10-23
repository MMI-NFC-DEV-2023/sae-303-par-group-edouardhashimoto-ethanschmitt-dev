<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import * as Plot from '@observablehq/plot'
import PlotFigure from '@/components/PlotFigure.js'
import departement25Doubs from '@/assets/departement-25-doubsGeojson.json'
import departement29Finistere from '@/assets/departement-29-finistereGeojson.json'
// import data25 from '@/assets/data25.json'
// import data29 from '@/assets/data29.json'
import { computed, ref, watch } from 'vue'
import * as d3 from 'd3'


const data29 = await fetch("data29.json").then((res) => res.json())
const data25 = await fetch("data25.json").then((res) => res.json())


// Mois de l'année
const monthIndex = [
  'Janvier',
  'Février',
  'Mars',
  'Avril',
  'Mai',
  'Juin',
  'Juillet',
  'Août',
  'Septembre',
  'Octobre',
  'Novembre',
  'Décembre'
]

// Année et mois sélectionnés
const selectedYear2 = ref(Math.min(...data25.map((d) => d.AAAAMM.slice(0, 4))))
const selectedMonth = ref(monthIndex[0])

// Calcul des températures moyennes par date
const temperature25 = computed(() =>
  data25.map((d) => {
    const year = d.AAAAMM.slice(0, 4)
    const month = d.AAAAMM.slice(4, 6) - 1
    return {
      date: new Date(year, month),
      temperature: (+d.TX + +d.TN) / 2,
      longitude: +d.LON,
      latitude: +d.LAT
    }
  })
)

const temperature29 = computed(() =>
  data29.map((d) => {
    const year = d.AAAAMM.slice(0, 4)
    const month = d.AAAAMM.slice(4, 6) - 1
    return {
      date: new Date(year, month),
      temperature: (+d.TX + +d.TN) / 2,
      longitude: +d.LON,
      latitude: +d.LAT
    }
  })
)

// Échelle de couleur pour la température
const colorScale = d3
  .scaleSequential()
  .domain(d3.extent(temperature25.value, (d) => d.temperature).reverse())
  .interpolator(d3.interpolateRdYlBu)

// Filtrer les températures par année et mois sélectionnés
const filteredTemperature25 = computed(() => {
  const filteredData = temperature25.value
    .filter(
      (d) =>
        d.date.getFullYear() === +selectedYear2.value &&
        d.date.getMonth() === monthIndex.indexOf(selectedMonth.value)
    )
    .filter((d) => d.temperature != null && !isNaN(d.temperature) && d.temperature !== 0)

  if (filteredData.length === 0) {
    console.warn('Aucune donnée disponible pour cette période.')
  }

  return filteredData
})

const filteredTemperature29 = computed(() => {
  const filteredData2 = temperature29.value
    .filter(
      (d) =>
        d.date.getFullYear() === +selectedYear2.value &&
        d.date.getMonth() === monthIndex.indexOf(selectedMonth.value)
    )
    .filter((d) => d.temperature != null && !isNaN(d.temperature) && d.temperature !== 0)

  if (filteredData2.length === 0) {
    console.warn('Aucune donnée disponible pour cette période.')
  }

  return filteredData2
})

// Options de graphique Plot
const plotOptions25 = computed(() => ({
  projection: {
    type: 'mercator',
    domain: departement25Doubs
  },
  marks: [
    Plot.geo(departement25Doubs, {
      fill: '#d4f0c0',
      stroke: '#666'
    }),
    ...(filteredTemperature25.value.length > 0
      ? [
          Plot.dot(filteredTemperature25.value, {
            x: 'longitude',
            y: 'latitude',
            fill: (d) => colorScale(d.temperature),
            r: 20,
            stroke: 'black',
            strokeWidth: 1
          }),
          Plot.text(filteredTemperature25.value, {
            x: 'longitude',
            y: 'latitude',
            text: (d) => d.temperature.toFixed(1) + '°C',
            dx: 35,
            dy: -25,
            fontSize: 20,
            fontWeight: 'bold',
            fill: 'black'
          })
        ]
      : [])
  ],
  width: 1000,
  height: 800,
  margin: 0,
  animation: true
}))

const plotOptions29 = computed(() => ({
  projection: {
    type: 'mercator',
    domain: departement29Finistere
  },
  marks: [
    Plot.geo(departement29Finistere, {
      fill: '#d4f0c0',
      stroke: '#666'
    }),
    ...(filteredTemperature29.value.length > 0
      ? [
          Plot.dot(filteredTemperature29.value, {
            x: 'longitude',
            y: 'latitude',
            fill: (d) => colorScale(d.temperature),
            r: 20,
            stroke: 'black',
            strokeWidth: 1
          }),
          Plot.text(filteredTemperature29.value, {
            x: 'longitude',
            y: 'latitude',
            text: (d) => d.temperature.toFixed(1) + '°C',
            dx: 35,
            dy: -25,
            fontSize: 20,
            fontWeight: 'bold',
            fill: 'black'
          })
        ]
      : [])
  ],
  width: 1000,
  height: 800,
  margin: 0,
  animation: true
}))

// Clé unique pour forcer la mise à jour du graphique
const plotKey = ref(0)

// Watch for changes in selectedYear2 and selectedMonth to trigger reactivity
watch([selectedYear2, selectedMonth], () => {
  plotKey.value += 1 // Incrémente la clé pour forcer la mise à jour
})
</script>

<template>
  <h1>Visualisation des températures dans le Doubs</h1>

  <label for="year-select">Sélectionnez une année : {{ selectedYear2 }}</label>
  <input
    type="range"
    v-model="selectedYear2"
    :min="Math.min(...data25.map((d) => d.AAAAMM.slice(0, 4)))"
    :max="Math.max(...data25.map((d) => d.AAAAMM.slice(0, 4)))"
    step="1"
  />

  <label for="month-select">Sélectionnez un mois :</label>
  <select v-model="selectedMonth" id="month-select">
    <option v-for="month in monthIndex" :key="month" :value="month">{{ month }}</option>
  </select>

  <div class=" lg:flex lg:justify-between">
    <PlotFigure :key="plotKey" :options="plotOptions25" />
    <PlotFigure :key="plotKey" :options="plotOptions29" />
  </div>
</template>

<style scoped>
.plot-container {
  display: flex;
  justify-content: space-between;
}
</style>
