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
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center mt-20 mb-6">Graphique</h1>
    <p class="mb-4">
      Les deux cartes interactives pour les départements 25 (Doubs) et 29 (Finistère) permettent de visualiser les températures moyennes enregistrées selon une année et un mois sélectionnés, de 1950 jusqu'en 2022. Chaque point sur la carte représente une localisation spécifique avec sa température moyenne. Les données de température sont affichées pour chaque région géographique, offrant une vue d'ensemble de la répartition des températures dans ces deux départements.
    </p>
    <h1 class="text-4xl font-bold text-center mb-6">Analyse</h1>
    <p class="mb-4">
      <strong>Département 25 (Doubs) :</strong><br>
      <strong>Valeurs observées :</strong> Les températures dans le Doubs varient généralement entre 15°C et 21°C en juin 2022, avec des zones plus chaudes vers l'ouest et le centre du département, tandis que les régions plus au sud montrent des températures plus fraîches.<br>
      <strong>Variation régionale :</strong> Les écarts de température peuvent être attribués à l'altitude et à la proximité des zones montagneuses dans le Doubs, où les températures sont plus fraîches, notamment au sud.<br>
      <strong>Évolution au fil des années :</strong> En analysant les tendances climatiques pour le Doubs, une augmentation des températures est probable, surtout durant les mois estivaux. Les températures moyennes au fil des années montrent une tendance à la hausse, avec des étés plus chauds et des hivers plus doux, conséquence directe du réchauffement climatique.
    </p>
    <p class="mb-4">
      <strong>Département 29 (Finistère) :</strong><br>
      <strong>Valeurs observées :</strong> Les températures dans le Finistère pour juin 2022 oscillent entre 15,4°C à 17.4°C, légèrement inférieures à celles du Doubs. La proximité de l'océan Atlantique joue un rôle majeur dans la régulation des températures, rendant les écarts saisonniers moins prononcés que dans les régions continentales.<br>
      <strong>Variation régionale :</strong> Les zones côtières du Finistère, exposées aux vents maritimes, sont légèrement plus fraîches, tandis que les zones intérieures, plus éloignées de l'influence océanique, enregistrent des températures plus élevées.<br>
      <strong>Évolution au fil des années :</strong> Comme dans le Doubs, le Finistère montre également une hausse des températures, bien que l'augmentation soit plus modérée en raison du climat océanique qui tempère les extrêmes. Toutefois, des étés plus chauds ont été enregistrés ces dernières années, un signe de la progression du réchauffement climatique même dans cette région.
    </p>
    <p class="mb-4">
      <strong>Visualisation :</strong> Les deux cartes utilisent une palette de couleurs codée pour illustrer les variations de température. Les températures plus basses sont représentées en rouge clair, tandis que les températures plus élevées sont en rouge foncé. Les points sont annotés avec les températures exactes, facilitant une comparaison visuelle rapide entre les différentes localisations des deux départements.
    </p>
    <p class="mb-4">
      <strong>Contexte Climatique et Comparaison :</strong><br>
      <strong>Doubs :</strong> Subissant un climat plus continental, le Doubs montre une amplitude thermique plus marquée avec des étés plus chauds et des hivers plus froids. Le réchauffement climatique se traduit par une augmentation notable des températures estivales et une réduction des épisodes froids extrêmes.<br>
      <strong>Finistère :</strong> Le Finistère, en revanche, est plus tempéré grâce à l'océan Atlantique, avec des étés doux et des hivers humides. Cependant, une tendance à l’augmentation des températures moyennes est également observable, affectant les écosystèmes côtiers et marins.
    </p>
    <p class="mb-4">
      <strong>Conclusion :</strong> Les deux départements montrent des tendances différentes dues à leurs géographies et climats respectifs, mais les deux suivent une augmentation progressive des températures liée au réchauffement climatique. Le Doubs, avec son climat continental, subit des variations plus marquées, tandis que le Finistère bénéficie d'un climat océanique qui amortit ces changements. Ces données renforcent l'idée que, bien que les effets du changement climatique varient localement, son impact est global.
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
.plot-container {
  display: flex;
  justify-content: space-between;
}
</style>
