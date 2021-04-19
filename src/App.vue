<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      app
    >
      <!--  -->
    </v-navigation-drawer>

    <v-app-bar app>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title>Spark Monitor</v-toolbar-title>
    </v-app-bar>

    <v-main>
      <overview></overview>
    </v-main>
  </v-app>
</template>

<script>
import Overview from './components/Overview';

export default {
  name: 'App',

 components: {
    Overview,
  },

  data: () => ({
    drawer: null
  }),
};
</script>
