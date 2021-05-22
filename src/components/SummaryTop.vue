<template>
 <v-row>
    <v-col cols="4">
        <v-card>
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                <v-list-item-icon class="text-left ma-0">
                  <v-icon color="green">mdi-clock</v-icon>
                </v-list-item-icon>
                {{ application_runtime }}
              </v-list-item-title>
              <v-list-item-subtitle>
                <b>id:</b> <a :href="spark_api_current_app" target="_blank">{{ application_id }}</a></v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>
        </v-card>
      </v-col>
      <v-col>
        <v-card>
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                <v-list-item-icon class="text-left ma-0">
                  <v-icon color="green">mdi-check-circle</v-icon>
                </v-list-item-icon>
                Jobs:
                {{ job_number }}
              </v-list-item-title>
              <v-list-item-subtitle>
                Number of completed Jobs</v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>
        </v-card>
      </v-col>
      <v-col>
        <v-card>
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                <v-list-item-icon class="text-left ma-0">
                  <v-icon color="green">mdi-check-circle</v-icon>
                </v-list-item-icon>
                Stages:
                {{ stage_number }}
              </v-list-item-title>
              <v-list-item-subtitle>
                Number of completed Stages</v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>
        </v-card>
      </v-col>
      <v-col>
        <v-card>
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                <v-list-item-icon class="text-left ma-0">
                  <v-icon color="green">mdi-check-circle</v-icon>
                </v-list-item-icon>
                Tasks:
                {{ task_number_total }}
              </v-list-item-title>
              <v-list-item-subtitle>
                Number of completed Tasks</v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>
        </v-card>
      </v-col>
 </v-row>
</template>

<script>
//import axios from "axios";

export default {
  name: "SummaryTop",
  data: () => ({
  
  }),
  props: [
      'application_runtime',
      'application_id',
      'spark_api_current_app',
      'jobs',
      'job_stages',
      ],
  computed: {
    stage_number() {
      return this.job_stages.length;
    },
    task_number_total() {
      let total_tasks = 0;
      for (var i = 0; i < this.job_stages.length; i++) {
        total_tasks = total_tasks + this.job_stages[i].numTasks;
      }
      return total_tasks;
    },
    job_number() {
      return this.jobs.length;
    },
}
}
</script>
