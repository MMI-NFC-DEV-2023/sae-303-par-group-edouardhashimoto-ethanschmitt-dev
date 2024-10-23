<!-- eslint-disable vue/multi-word-component-names -->
<template>
  <div>
    <div>
      <label for="year-select">Sélectionnez une année</label>
      <input
        id="year-select"
        type="range"
        :min="Math.min(...yearsAvailable)"
        :max="Math.max(...yearsAvailable)"
        v-model="selectedYear2"
        step="1"
      />
      <span>{{ selectedYear2 }}</span>
    </div>

    <div>
      <label for="month-select">Sélectionnez un mois</label>
      <select v-model="selectedMonth">
        <option v-for="(month, index) in monthIndex" :key="index">{{ month }}</option>
      </select>
    </div>

    <div class="plot-container">
      <div ref="plotContainer1" class="plot"></div>
      <div ref="plotContainer2" class="plot"></div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import * as Plot from '@observablehq/plot';
import departement25Doubs from '@/assets/departement-25-doubsGeojson.json';
import departement29Finistere from '@/assets/departement-29-finistereGeojson.json';
import data25 from '@/assets/data25.json';
import data29 from '@/assets/data29.json';

export default {
  data() {
    return {
      temperature25: this.formatData(data25),
      temperature29: this.formatData(data29),
      monthIndex: [
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
      ],
      selectedYear2: new Date().getFullYear(),
      selectedMonth: 'Janvier',
      yearsAvailable: []
    };
  },
  mounted() {
    this.yearsAvailable = Array.from(
      new Set(this.temperature25.map((d) => d.date.getFullYear()))
    ).sort();
    this.drawPlot();
  },
  watch: {
    selectedYear2() {
      this.drawPlot();
    },
    selectedMonth() {
      this.drawPlot();
    }
  },
  methods: {
    formatData(data) {
      return data.map((d) => {
        const year = d.AAAAMM.slice(0, 4);
        const month = d.AAAAMM.slice(4, 6) - 1; // Convertir le mois en index (0-11)
        return {
          date: new Date(year, month),
          temperature: (+d.TX + +d.TN) / 2,
          longitude: +d.LON,
          latitude: +d.LAT
        };
      });
    },
    getFilteredData(temperatureData) {
      return temperatureData
        .filter(
          (d) =>
            d.date.getFullYear() === this.selectedYear2 &&
            d.date.getMonth() === this.monthIndex.indexOf(this.selectedMonth)
        )
        .filter((d) => d.temperature != null && !isNaN(d.temperature) && d.temperature !== 0);
    },
    drawPlot() {
      const filteredTemperature25 = this.getFilteredData(this.temperature25);
      const filteredTemperature29 = this.getFilteredData(this.temperature29);

      const colorScale25 = d3
        .scaleSequential()
        .domain(d3.extent(filteredTemperature25, (d) => d.temperature).reverse())
        .interpolator(d3.interpolateRdYlBu);

      const colorScale29 = d3
        .scaleSequential()
        .domain(d3.extent(filteredTemperature29, (d) => d.temperature).reverse())
        .interpolator(d3.interpolateRdYlBu);

      // Plot pour Doubs
      const plot1 = Plot.plot({
        projection: {
          type: 'mercator',
          domain: departement25Doubs
        },
        marks: [
          Plot.geo(departement25Doubs, {
            fill: '#d4f0c0',
            stroke: '#666'
          }),
          ...(filteredTemperature25.length > 0
            ? [
                Plot.dot(filteredTemperature25, {
                  x: 'longitude',
                  y: 'latitude',
                  fill: (d) => colorScale25(d.temperature),
                  r: 10,
                  stroke: 'black',
                  strokeWidth: 1
                }),
                Plot.text(filteredTemperature25, {
                  x: 'longitude',
                  y: 'latitude',
                  text: (d) => d.temperature.toFixed(1) + '°C',
                  dx: 15,
                  dy: -15,
                  fontSize: 12,
                  fontWeight: 'bold',
                  fill: 'black'
                })
              ]
            : [])
        ],
        width: 1000,
        height: 500,
        margin: 0,
        animation: true
      });

      // Plot pour Finistère
      const plot2 = Plot.plot({
        projection: {
          type: 'mercator',
          domain: departement29Finistere
        },
        marks: [
          Plot.geo(departement29Finistere, {
            fill: '#d4f0c0',
            stroke: '#666'
          }),
          ...(filteredTemperature29.length > 0
            ? [
                Plot.dot(filteredTemperature29, {
                  x: 'longitude',
                  y: 'latitude',
                  fill: (d) => colorScale29(d.temperature),
                  r: 10,
                  stroke: 'black',
                  strokeWidth: 1
                }),
                Plot.text(filteredTemperature29, {
                  x: 'longitude',
                  y: 'latitude',
                  text: (d) => d.temperature.toFixed(1) + '°C',
                  dx: 15,
                  dy: -15,
                  fontSize: 12,
                  fontWeight: 'bold',
                  fill: 'black'
                })
              ]
            : [])
        ],
        width: 1000,
        height: 500,
        margin: 0,
        animation: true
      });

      this.$refs.plotContainer1.innerHTML = '';
      this.$refs.plotContainer1.appendChild(plot1);

      this.$refs.plotContainer2.innerHTML = '';
      this.$refs.plotContainer2.appendChild(plot2);
    }
  }
};
</script>

<style>
.plot-container {
  display: flex;
  justify-content: space-between; /* Optionnel : espace entre les éléments */
}

.plot {
  flex: 1; /* Les deux éléments prendront une largeur égale */
  margin: 0 10px; /* Optionnel : marge entre les éléments */
}
</style>
