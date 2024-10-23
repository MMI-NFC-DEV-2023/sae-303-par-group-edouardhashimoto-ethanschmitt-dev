<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import * as Plot from '@observablehq/plot'
import PlotFigure from '@/components/PlotFigure.js'
import data25 from '@/assets/data25.json'
import data29 from '@/assets/data29.json'
import { computed, ref, watch } from 'vue'

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
  <h1>Plot + Vue</h1>

  <label for="event-select">Événement Météorologique :</label>
  <select v-model="selectedEvent" id="event-select">
    <option v-for="event in events" :key="event" :value="event">{{ event }}</option>
  </select>

  <PlotFigure :key="plotKey" :options="plotOptions" />
</template>