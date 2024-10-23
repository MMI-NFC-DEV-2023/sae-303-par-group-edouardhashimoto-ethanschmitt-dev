<template>
  <Header />
  <main class="container mx-auto p-4">
    <h1 class="text-6xl flex justify-center items-center mb-20 mt-10">Changement climatique</h1>
    <section
      class="bg-white p-6 rounded-lg shadow-md mb-6 transition-transform transform hover:scale-105"
    >
      <h2 class="text-2xl font-bold mb-4">Contexte du projet</h2>
      <p class="mb-4">
        Description rapide des utilisateurs, de la problématique, de l'état de l'art et des besoins.
        Dans son dernier rapport (2023), le Giec estime à 1,1°C l’augmentation de la température
        moyenne mondiale depuis la fin du XIXe siècle, qu’il attribue largement aux émissions de gaz
        à effet de serre générées par les activités humaines. Il formule également, à partir de
        différents scénarios d’émissions, des projections qui prévoient une élévation de température
        de +1,4°C (scénario le plus optimiste) à +4,4°C (scénario le plus pessimiste) d’ici 2100.
        Les analyses du Giec sont établies à un niveau mondial, et pour de grandes régions comme
        l’Europe ou le bassin méditerranéen. Des études climatiques sur des échelles plus réduites
        sont encore rares.
      </p>
    </section>
    <section
      class="bg-white p-6 rounded-lg shadow-md mb-6 transition-transform transform hover:scale-105"
    >
      <h2 class="text-2xl font-bold mb-4">Objectifs du projet</h2>
      <p>
        L'objectif général du projet et les impacts recherchés. Les travaux réalisés dans le cadre
        de ce projet permettront : A chaque citoyen de mieux comprendre les enjeux liés au
        changement climatique et de participer au débat public ; Aux décideurs publics (nationaux et
        locaux) de mieux penser les stratégies d’atténuation (maîtrise des émissions de gaz à effet
        de serre) et d’adaptation (limitation des impacts du changement climatique et des dommages
        associés sur les sociétés humaines et les écosystèmes) grâce à des connaissances
        territorialisées.
      </p>
    </section>
    <div class="flex flex-wrap justify-center space-x-4">
      <section
        class="relative bg-white p-6 rounded-lg shadow-md mb-6 transition-transform transform hover:scale-105 max-w-lg"
      >
        <div
          class="absolute inset-0 bg-cover bg-center opacity-50"
          style="
            background-image: url('/public/img/close-up-thermometer-showing-high-temperature.jpg');
          "
        ></div>
        <div class="relative z-10 bg-white bg-opacity-75 p-4 rounded">
          <h2 class="text-2xl font-bold mb-4">Présentation du Graphique 1</h2>
          <p class="mb-4">
            Un graphique montreant la température maximale par année pour les départements 25 et
            29. Il permet de visualiser les tendances de température au fil des années.
          </p>
          <RouterLink
            to="/graphique1"
            class="inline-block bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-700 transition duration-300"
          >
            Voir le Graphique
          </RouterLink>
        </div>
      </section>
      <section
        class="relative bg-white p-6 rounded-lg shadow-md mb-6 transition-transform transform hover:scale-105 max-w-lg"
      >
        <div
          class="absolute inset-0 bg-cover bg-center opacity-50"
          style="background-image: url('/public/img/weather-effects-collage-concept.jpg')"
        ></div>
        <div class="relative z-10 bg-white bg-opacity-75 p-4 rounded">
          <h2 class="text-2xl font-bold mb-4">Présentation du Graphique 2</h2>
          <p class="mb-4">
            Ce graphique présente les événements météorologiques passés depuis 1950 à 2022 pour les
            départements 25 et 29. Il permet de visualiser les tendances et les occurrences de ces
            événements au fil des années.
          </p>
          <RouterLink
            to="/graphique2"
            class="inline-block bg-green-500 text-white px-4 py-2 rounded hover:bg-green-700 transition duration-300"
          >
            Voir le Graphique
          </RouterLink>
        </div>
      </section>
    </div>
    <div class="flex flex-wrap justify-center space-x-4">
      <section
        class="relative bg-white p-6 rounded-lg shadow-md mb-6 transition-transform transform hover:scale-105 max-w-lg"
      >
        <div
          class="absolute inset-0 bg-cover bg-center opacity-50"
          style="background-image: url('/public/img/thunderstorm-city.jpg')"
        ></div>
        <div class="relative z-10 bg-white bg-opacity-75 p-4 rounded">
          <h2 class="text-2xl font-bold mb-4">Présentation du Graphique 3</h2>
          <p class="mb-4">
            Ce graphique montre une carte des départements 25 et 29 et affiche les températures
            moyennes pour chaque mois et année. Il permet de visualiser les variations de
            température sur une carte interactive.
          </p>
          <RouterLink
            to="/graphique3"
            class="inline-block bg-red-500 text-white px-4 py-2 rounded hover:bg-red-700 transition duration-300"
          >
            Voir le Graphique
          </RouterLink>
        </div>
      </section>
      <section
        class="relative bg-white p-6 rounded-lg shadow-md mb-6 transition-transform transform hover:scale-105 max-w-lg"
      >
        <div
          class="absolute inset-0 bg-cover bg-center opacity-50"
          style="background-image: url('/public/img/dolomites-mountains-with-snow.jpg')"
        ></div>
        <div class="relative z-10 bg-white bg-opacity-75 p-4 rounded">
          <h2 class="text-2xl font-bold mb-4">Présentation du Graphique 4</h2>
          <p class="mb-4">
            Ce graphique présente le nombre de jours d'enneigement dans le département 25 par année
            et mois. Il permet de visualiser les tendances d'enneigement au fil des années.
          </p>
          <RouterLink
            to="/graphique4"
            class="inline-block bg-purple-500 text-white px-4 py-2 rounded hover:bg-purple-700 transition duration-300"
          >
            Voir le Graphique
          </RouterLink>
        </div>
      </section>
    </div>
    <RouterView v-slot="{ Component }">
      <Suspense>
        <component :is="Component" :key="$route.path" />
      </Suspense>
    </RouterView>
  </main>
</template>
