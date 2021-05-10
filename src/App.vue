<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      app
    >
      <!--  -->
    </v-navigation-drawer>

    <v-app-bar app color="deep-purple accent-4" dark> 
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title>Spark Live Activity Monitor</v-toolbar-title>
      <v-spacer></v-spacer>

       <v-chip
      class="ma-2"
      :color=status_color
      text-color="white"
    >{{status_text}}
    </v-chip>
    </v-app-bar>

    <v-main>
      <overview v-on:heartbeat="aliveUpdate"></overview>
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
    drawer: null,
    status_text: "not connected",
    status_color: "red"
  }),
  methods: {
    aliveUpdate(value){
      console.log("Spark heart beat status: ",value)
      if(value=="OK"){
        this.status_text="connected"
        this.status_color="green"
      }
      else{
        this.status_text="not connected"
        this.status_color="red"
      }
    }
  }
};
</script>
