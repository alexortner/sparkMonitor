<template>
  <v-card class="mx-auto" max-width="600">
    <v-list-item two-line>
      <v-list-item-content>
        <v-list-item-title class="headline"> Job Details </v-list-item-title>
        <v-list-item-subtitle>
          <b>Active Job ({{ active_job.jobId }}):</b>
          {{ active_job.name }}</v-list-item-subtitle
        >
        <v-list-item-subtitle>
          <b>Active Stage ({{ active_stage.stageId }}):</b>
          {{ active_stage.name }}</v-list-item-subtitle
        >
        <v-list-item-subtitle>
          <b>Runtime Active Stage ({{ active_stage.stageId }}):</b>
          {{ current_stage_runtime }}</v-list-item-subtitle
        >
      </v-list-item-content>
    </v-list-item>

    <v-divider></v-divider>

    <v-card-subtitle class="pb-0"> Stages </v-card-subtitle>
    <v-card-text class="text--primary pb-0">
      <v-progress-linear v-model="stage_completion_percent" height="25">
        <strong
          >{{ active_job.numCompletedStages + active_job.numSkippedStages }}/{{
            active_job.stageIds.length
          }}
          ({{ active_job.numActiveStages }} running,
          {{ active_job.numFailedStages }} failed)</strong
        >
      </v-progress-linear>
    </v-card-text>

    <v-card-subtitle class="pb-0 pt-2"> Tasks </v-card-subtitle>
    <v-card-text class="text--primary">
      <v-progress-linear v-model="task_completion_percent" height="25">
        <strong
          >{{ active_stage.numCompleteTasks }}/{{ active_stage.numTasks }} ({{
            active_stage.numActiveTasks
          }}
          running, {{ active_stage.numFailedTasks }} failed)</strong
        >
      </v-progress-linear>
    </v-card-text>
  </v-card>
</template>
<script>
export default {
  name: "SummaryJobs",
  data: () => ({
  
  }),
  props: [
      'jobs',
      'job_stages'
      ],

  methods: {
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
  computed: {
    active_job() {
      return this.jobs.filter((entry) => entry.status == "RUNNING")[0];
    },
    active_stage() {
      return this.job_stages.filter((entry) => entry.status == "ACTIVE")[0];
    },
    current_stage_runtime() {
      return this.calculateRuntime(this.active_stage.firstTaskLaunchedTime);
    },
    task_completion_percent() {
      return Math.ceil(
        (Number(this.active_stage.numCompleteTasks) /
          Number(this.active_stage.numTasks)) *
          100
      );
    },
    stage_completion_percent() {
      return Math.ceil(
        (Number(
          this.active_job.numCompletedStages + this.active_job.numSkippedStages
        ) /
          Number(this.active_job.stageIds.length)) *
          100
      );
    },
  }
}
</script>