<template>
  <v-container>
    <v-row>
      <v-col cols="6">
        <v-card class="mx-auto" max-width="600">
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                Application Info:
              </v-list-item-title>

              <v-list-item-subtitle
                ><b>name:</b> {{ application_name }}</v-list-item-subtitle
              >
              <v-list-item-subtitle
                ><b>id:</b> {{ application_id }}</v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>

          <v-list-item dense>
            <v-list-item-title>Job Uptime:</v-list-item-title>

            <v-list-item-subtitle>{{
              application_runtime
            }}</v-list-item-subtitle>
            <v-list-item-icon class="text-right">
              <v-icon>mdi-clock</v-icon>
            </v-list-item-icon>
          </v-list-item>

          <v-divider></v-divider>

          <v-list class="transparent" dense>
            <v-list-item>
              <v-list-item-title>Jobs:</v-list-item-title>
              <v-list-item-subtitle>{{ job_number }}</v-list-item-subtitle>
            </v-list-item>
            <v-list-item>
              <v-list-item-title>Stages:</v-list-item-title>
              <v-list-item-subtitle>{{ stage_number }}</v-list-item-subtitle>
            </v-list-item>
            <v-list-item>
              <v-list-item-title>Tasks:</v-list-item-title>
              <v-list-item-subtitle>{{ task_number }}</v-list-item-subtitle>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>

      <!-- Right Card -->
      <v-col class="mb-4">
        <v-card class="mx-auto" max-width="600">
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                Current Stage Details
              </v-list-item-title>
              <v-list-item-subtitle>
                <b>name:</b> {{ active_stage.name }}</v-list-item-subtitle
              >
              <v-list-item-subtitle
                ><b>id:</b> {{ active_stage.stageId }}</v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>

          <v-list-item dense>
            <v-list-item-title>Stage Runtime:</v-list-item-title>

            <v-list-item-subtitle>{{
              current_stage_runtime
            }}</v-list-item-subtitle>
            <v-list-item-icon class="text-right">
              <v-icon>mdi-clock</v-icon>
            </v-list-item-icon>
          </v-list-item>

          <v-divider></v-divider>

          <v-card-subtitle class="pb-0"> Stages </v-card-subtitle>
          <v-card-text class="text--primary pb-0">
            <v-progress-linear v-model="stage_completion_percent" height="25">
              <strong
                >{{ jobs[0].numCompletedStages }}/{{ stage_number }} ({{
                  jobs[0].numActiveStages
                }}
                running, {{ jobs[0].numFailedStages }} failed)</strong
              >
            </v-progress-linear>
          </v-card-text>

          <v-card-subtitle class="pb-0 pt-2"> Tasks </v-card-subtitle>
          <v-card-text class="text--primary">
            <v-progress-linear v-model="task_completion_percent" height="25">
              <strong
                >{{ active_stage.numCompleteTasks }}/{{
                  active_stage.numTasks
                }}
                ({{ active_stage.numActiveTasks }} running,
                {{ active_stage.numFailedTasks }} failed)</strong
              >
            </v-progress-linear>
          </v-card-text>
        </v-card>
      </v-col>

      <v-col class="mb-5" cols="12">
        <h1>Details</h1>
        <v-expansion-panels>
          <v-expansion-panel>
            <v-expansion-panel-header> Stage Details </v-expansion-panel-header>
            <v-expansion-panel-content>
              Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
              eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut
              enim ad minim veniam, quis nostrud exercitation ullamco laboris
              nisi ut aliquip ex ea commodo consequat.
            </v-expansion-panel-content>
          </v-expansion-panel>
          <v-expansion-panel>
            <v-expansion-panel-header>
              <span style="width: 10%">Task Details </span>
              <v-progress-linear
                v-model="task_completion_percent"
                height="25"
                class="mr-5"
              >
                <strong
                  >{{ active_stage.numCompleteTasks }}/{{
                    active_stage.numTasks
                  }}
                  ({{ active_stage.numActiveTasks }} running,
                  {{ active_stage.numFailedTasks }} failed)</strong
                >
              </v-progress-linear>
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <div>
                <v-data-table
                  dense
                  :headers="headers"
                  :items="taskFilter"
                  item-key="taskId"
                  :sort-by="['status', 'index']"
                  :sort-desc="[false, false]"
                  multi-sort
                  class="elevation-1"
                  :search="search"
                  disable-pagination
                >
                  <template v-slot:item.status="{ item }">
                    <v-chip :color="getColor(item.status)" dark small>
                      {{ item.status }}
                    </v-chip></template
                  >
                  <template v-slot:top>
                    <v-toolbar flat>
                      <!--<v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                      ></v-text-field>-->
                      <v-spacer></v-spacer>
                      <v-btn-toggle v-model="filter">
                        <v-btn rounded color="orange" dark small class="ml-2">
                          RUNNING
                        </v-btn>
                        <v-btn class="ml-2" rounded color="green" dark small>
                          SUCCESS
                        </v-btn>
                        <v-btn class="ml-2" rounded color="red" dark small>
                          FAILED
                        </v-btn>
                      </v-btn-toggle>
                    </v-toolbar>
                  </template>
                </v-data-table>
              </div>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-col>

      <v-col class="mb-5" cols="12">
        JSON RAW
        <pre>{{ application_id }}</pre>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  name: "Overview",

  data: () => ({
    application_id: null,
    application_name: "",
    application_starttime: null,
    application_runtime: "",
    application_json: null,
    job_stages: [],
    jobs: [],
    tasklist: [],
    search: "",
    filter: "",
  }),
  created() {
    this.apiCallWrapper();
    setInterval(this.apiCallWrapper, 3000);
  },
  methods: {
    async apiCallWrapper() {
      try {
        await this.getApplicationInfo();
        await this.getJopDetails();
        await this.getStageDetails();
        await this.getCurrentTaskList();
      } catch (e) {
        console.log(e);
      }
    },

    async getApplicationInfo() {
      const response = await axios.get(
        "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications",
        {
          headers: {
            "Access-Control-Allow-Credentials": "true",
            "Content-Type": "application/json",
          },
        }
      );

      this.application_id = response.data[0].id;
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

    async getJopDetails() {
      const response = await axios.get(
        "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications/".concat(
          this.application_id,
          "/jobs"
        ),
        {
          headers: {
            "Access-Control-Allow-Credentials": "true",
            "Content-Type": "application/json",
          },
        }
      );

      this.jobs = response.data;
      this.jobs.sort((a, b) => (a.jobID > b.jobID ? 1 : -1));
    },

    async getStageDetails() {
      const response = await axios.get(
        "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications/".concat(
          this.application_id,
          "/stages"
        ),
        {
          headers: {
            "Access-Control-Allow-Credentials": "true",
            "Content-Type": "application/json",
          },
        }
      );

      this.job_stages = response.data;
      this.job_stages.sort((a, b) => (a.stageID > b.stageID ? 1 : -1));
    },

    async getCurrentTaskList() {
      let current_stage = this.job_stages.filter(
        (entry) => entry.status == "ACTIVE"
      )[0].stageId;
      let current_stage_attempt = this.job_stages.filter(
        (entry) => entry.status == "ACTIVE"
      )[0].attemptId;
      const response = await axios.get(
        "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications/".concat(
          this.application_id,
          "/stages/",
          current_stage,
          "/",
          current_stage_attempt,
          "/taskList",
          "?length=200"
        ),
        {
          headers: {
            "Access-Control-Allow-Credentials": "true",
            "Content-Type": "application/json",
          },
        }
      );
      this.tasklist = response.data;
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
    getColor(status) {
      if (status == "FAILED") return "red";
      else if (status == "RUNNING") return "orange";
      else return "green";
    },
  },

  computed: {
    stage_number() {
      return this.job_stages.length;
    },
    job_number() {
      return this.jobs.length;
    },
    task_number() {
      return this.jobs[0].numTasks;
    },
    active_job() {
      return this.jobs.filter((entry) => entry.status == "RUNNING");
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
        (Number(this.jobs[0].numCompletedStages) / Number(this.stage_number)) *
          100
      );
    },
    taskFilter() {
      const filter = this.filter;
      let taskListFiltered = [];
      let filterstring = "";

      if (filter == "0") {
        filterstring = "RUNNING";
      } else if (filter == "1") {
        filterstring = "SUCCESS";
      } else if (filter == "2") {
        filterstring = "FAILED";
      } else {
        filterstring = "";
      }

      console.log("FILTER ", filterstring);
      if (filterstring != "") {
        taskListFiltered = this.tasklist.filter(
          (row) => row.status == filterstring
        );
      } else {
        taskListFiltered = this.tasklist;
      }

      var taskListFiltered2 = taskListFiltered.map(function (currentObject) {
        currentObject["runtime"]=currentObject.duration/1000/60
        return currentObject
});
console.log(taskListFiltered2)

      return taskListFiltered;
    },
    headers() {
      return [
        {
          text: "Task ID",
          align: "start",
          sortable: true,
          value: "index",
        },
        {
          text: "Task Run ID",
          sortable: true,
          value: "taskId",
        },
        { text: "Attempt", value: "attempt" },
        { text: "Executor", value: "executorId" },
        { text: "Host", value: "host" },
        { text: "Runtime", value: "duration" },
        { text: "Status", value: "status" },
      ];
    },
  },
};
</script>
