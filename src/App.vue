<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer" app>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title class="title"> Boards </v-list-item-title>
          <v-list-item-subtitle>
            Detailed view the Spark Job
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list dense nav>
        <v-list-item>
          <v-list-item-icon>
            <v-icon>mdi-view-dashboard</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Overview Dashboard</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app color="deep-purple accent-4" dark>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title>Spark Live Activity Monitor</v-toolbar-title>
      <v-spacer></v-spacer>

      <v-btn
        color="secondary"
        class="mr-1"
        small
        href="http://spark-ui.k8s-prod1.local.parcit/jobs/"
        target="_blank"
      >
        Spark UI</v-btn
      >
      <v-btn
        color="secondary"
        small
        :href="spark_api_current_app"
        target="_blank"
      >
        Spark API</v-btn
      >

      <v-chip class="ma-2" :color="status_color" text-color="white"
        >{{ status_text }}
      </v-chip>
    </v-app-bar>

    <v-main>
      <Dashboard v-if="application_id !== null"
        :application_id="application_id"
        :spark_api_current_app="spark_api_current_app"
        :application_runtime="application_runtime"
        :application_name="application_name"
      ></Dashboard>
    </v-main>
  </v-app>
</template>

<script>
import Dashboard from "./views/Dashboard";
import axios from "axios";

export default {
  name: "App",

  components: {
    Dashboard,
  },

  data: () => ({
    spark_api: "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications",
    drawer: false,
    status_text: "not connected",
    status_color: "red",
    startTime: 0,
    application_id: null,
    spark_api_current_app: "",
    application_name: "",
    application_starttime: null,
    application_runtime: "",
    interval: null
  }),
  created() {
    this.getCurrentTaskList()
    this.interval=setInterval(this.apiCallWrapper, 3000);
  },
  destroyed(){
    clearInterval(this.interval)
  },

  methods: {
    async apiCallWrapper() {
      try {
        //await this.getRancherAPI();
        this.setStartTime();
        await this.getApplicationInfo();
        this.status_text = "connected";
        this.status_color = "green";
      } catch (e) {
        console.log("Spark API connection Error: ", e);
        this.status_text = "not connected";
        this.status_color = "red";
      }
    },

    setStartTime() {
      if (this.startTime == 0) {
        this.startTime = new Date().getTime();
      }
    },

    async getApplicationInfo() {
      const response = await axios.get(this.spark_api, {
        headers: {
          "Access-Control-Allow-Credentials": "true",
          "Content-Type": "application/json",
        },
      });
      this.application_id = response.data[0].id;
      this.spark_api_current_app = this.spark_api + "/" + response.data[0].id;
      this.application_name = response.data[0].name;
      this.application_starttime =
        response.data[0].attempts[
          response.data[0].attempts.length - 1
        ].startTime;

      this.application_runtime = this.calculateRuntime(
        this.application_starttime
      );
      //console.log("current runtime: ", this.application_runtime)

      return response.data[0].id;
    },

    calculateRuntime(start) {
      let now = new Date();
      // calculate runtime
      let starttime = new Date(
        start.substr(0, 4),
        (Number(start.substr(5, 2)) - 1).toString(),
        start.substr(8, 2),
        (Number(start.substr(11, 2)) + 2).toString(),
        start.substr(14, 2),
        start.substr(17, 2)
      );
      let difference = new Date(now - starttime);
      const seconds = Math.floor((difference / 1000) % 60).toString();
      const minutes = Math.floor((difference / 1000 / 60) % 60).toString();
      const hours = Math.floor((difference / 1000 / 3600) % 24).toString();

      return hours.concat(":", minutes, ":", seconds);
    },
  },
};
</script>
