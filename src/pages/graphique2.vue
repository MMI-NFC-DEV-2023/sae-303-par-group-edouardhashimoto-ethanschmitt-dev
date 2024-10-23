<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import * as Plot from '@observablehq/plot'
import PlotFigure from '@/components/PlotFigure.js'
// import data25 from '@/assets/data25.json'
// import data29 from '@/assets/data29.json'
import { computed, ref, watch } from 'vue'

const data29 = await fetch('data29.json').then((res) => res.json())
const data25 = await fetch('data25.json').then((res) => res.json())

// Liste des événements disponibles
const events = ['NBJGREL', 'NBJORAG', 'NBJBROU']

// Événement sélectionné (NBJORAG par défaut)
const selectedEvent = ref('NBJORAG')

// Calcul des totaux pour chaque département
const totalEventDays25 = computed(() =>
  data25.reduce((sum, d) => sum + (+d[selectedEvent.value] || 0), 0)
)

const totalEventDays29 = computed(() =>
  data29.reduce((sum, d) => sum + (+d[selectedEvent.value] || 0), 0)
)

// Créer les données de graphique
const plotData = computed(() => [
  { name: 'Département 25', value: totalEventDays25.value, color: 'steelblue' },
  { name: 'Département 29', value: totalEventDays29.value, color: 'orange' }
])

// Options pour Plot
const plotOptions = computed(() => ({
  marks: [
    Plot.barY(plotData.value, {
      x: 'name',
      y: 'value',
      fill: (d) => d.color
    })
  ],
  y: { label: 'Nombre total de jours' },
  x: { domain: ['Département 25', 'Département 29'] }
}))

// Clé unique pour forcer la mise à jour du graphique
const plotKey = ref(0)

// Watch for changes in selectedEvent to trigger reactivity
watch(selectedEvent, () => {
  plotKey.value += 1 // Incrémente la clé pour forcer la mise à jour
})
</script>

<template>
  <div class="flex justify-center items-center">
    <div class="container mx-auto p-4">
      <div class="flex flex-col items-center">
        <label for="event-select" class="mb-2">Événement Météorologique :</label>
        <select v-model="selectedEvent" id="event-select" class="mb-4">
          <option v-for="event in events" :key="event" :value="event">{{ event }}</option>
        </select>
      </div>

      <div class="flex justify-center">
        <PlotFigure :key="plotKey" :options="plotOptions" />
      </div>
    </div>
  </div>
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center mt-20 mb-6">Graphique</h1>
    <p class="mb-4">
      Le graphique montre une comparaison du nombre total de jours liés à un événement
      météorologique spécifique comme le nombre de jours de brouillard, d'orage et de grêle entre
      deux départements français : le Département 25 et le Département 29.
    </p>
    <h1 class="text-4xl font-bold text-center mb-6">Analyse</h1>
    <p class="mb-4">
      Les valeurs observées : Le Département 25 (en bleu) a un total de 907 jours pour l'événement
      météorologique sélectionné. Le Département 29 (en orange) a un total beaucoup plus élevé, avec
      6 211 jours.
    </p>
    <p class="mb-4">
      Visualisation : La barre du Département 29 est considérablement plus haute que celle du
      Département 25, reflétant l'énorme écart entre les deux.
    </p>
    <p class="mb-4">
      Répartition des événements : Ces données pourraient indiquer une fréquence beaucoup plus
      élevée de l'événement météorologique sélectionné dans le Département 29 par rapport au
      Département 25. Par exemple, si l'événement sélectionné est lié aux orages, il est clair que
      le Département 29 connaît plus de jours orageux que le Département 25.
    </p>
    <p class="mb-4">
      Dans le cadre du projet, qui vise à analyser les changements climatiques en France à
      différentes échelles, cette visualisation peut illustrer des différences locales importantes
      dans la répartition des événements climatiques spécifiques (ici un phénomène météorologique).
      Ce type d'analyse permet aux citoyens et aux décideurs de mieux comprendre comment certains
      départements sont plus affectés que d'autres par des événements météorologiques particuliers,
      potentiellement liés au réchauffement climatique.
    </p>
    <p class="text-sm text-gray-500">Source</p>
    <p class="text-sm text-blue-500 hover:underline transition duration-300">
      <a href="https://defis.data.gouv.fr/datasets/6569b3d7d193b4daf2b43edc" target="_blank">
        https://defis.data.gouv.fr/datasets/6569b3d7d193b4daf2b43edc
      </a>
    </p>
    
    <div class="flex justify-end mt-5">
      <router-link to="/graphique3">
        <button
          class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-700 transition duration-300"
        >
          Aller à Graphique 3
        </button>
      </router-link>
    </div>

  </div>
</template>
