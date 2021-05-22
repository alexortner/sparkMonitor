<template>
<div>
  <ul id="example-1">
  <li v-for="item in task_list_raw" :key="item.index">
    {{ item.taskId }}
  </li>
</ul>

 <!--
<v-expansion-panels>
  <v-expansion-panel>
    <v-expansion-panel-header>
      <span style="width: 15%">Task Details </span>
      <v-progress-linear
        v-model="task_completion_percent"
        height="25"
        class="mr-5"
      >
        <strong
          >{{ active_stage.numCompleteTasks }}/{{ active_stage.numTasks }} ({{
            active_stage.numActiveTasks
          }}
          running, {{ active_stage.numFailedTasks }} failed)</strong
        >
      </v-progress-linear>
    </v-expansion-panel-header>
    <v-expansion-panel-content>
     <div>
        <v-data-table
          dense
          :headers="headers"
          :items="task_list_filtered"
          item-key="taskId"
          :sort-by="['status', 'index']"
          :sort-desc="[false, false]"
          multi-sort
          class="elevation-1"
          :search="search"
        >
          <template v-slot:item.status="{ item }">
            <v-tooltip left v-if="item.status === 'FAILED'" max-width="300">
              <template v-slot:activator="{ on, attrs }">
                <v-chip
                  v-bind="attrs"
                  v-on="on"
                  :color="getColor2(item.status)"
                  dark
                  small
                >
                  {{ item.status }}
                </v-chip>
              </template>
              <span>{{ item.errorMessage }}</span>
            </v-tooltip>
            <v-chip
              v-else
              v-bind="attrs"
              v-on="on"
              :color="getColor2(item.status)"
              dark
              small
            >
              {{ item.status }}
            </v-chip>
          </template>
          <template v-slot:top>
            <v-toolbar flat>
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
  </v-expansion-panels>-->
  </div>
</template>
<script>
import axios from "axios";


export default {
  name: "DetailsTask",
  data: () => ({
      filter: "",
      search: "",
      interval: null,
      task_list_raw: null
  }),
  props: [
    "spark_api", 
    "application_id",
    "job_stages"
    ],
  
  created() {
    this.getCurrentTaskList()
    this.interval=setInterval(this.apiCallWrapper, 3000);
  },
  destroyed(){
    clearInterval(this.interval)
  },

  methods: {
     async getCurrentTaskList() {
      let current_stage = this.job_stages.filter(
        (entry) => entry.status == "ACTIVE"
      )[0].stageId;
      let current_stage_attempt = this.job_stages.filter(
        (entry) => entry.status == "ACTIVE"
      )[0].attemptId;
      const response = await axios.get(
        this.spark_api.concat(
          "/",
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
      this.task_list_raw = response.data;
    },

    getColor2(status) {
      if (status == "FAILED") return "red";
      else if (status == "RUNNING") return "orange";
      else return "green";
    },

     task_list_filtered2(task_list_raw,filter) {
      //const filter = this.filter;
      if(task_list_raw===null){
        return []
      }
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

      if (filterstring != "") {
        taskListFiltered = task_list_raw.filter(
          (row) => row.status == filterstring
        );
      } else {
        taskListFiltered = task_list_raw;
      }
      console.log("HHHH",taskListFiltered)

      taskListFiltered = taskListFiltered.map(function (currentObject) {
        currentObject["runtime"] =
          Math.ceil(currentObject.duration / 1000 / 60) + " min";
        return currentObject;
      });

      return taskListFiltered;
    },

  },
  computed: {
    active_stage() {
      return this.job_stages.filter((entry) => entry.status == "ACTIVE")[0];
    },

    task_completion_percent() {
      return Math.ceil(
        (Number(this.active_stage.numCompleteTasks) /
          Number(this.active_stage.numTasks)) *
          100
      );
    },

    task_list_filtered() {
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

      if (filterstring != "") {
        taskListFiltered = this.task_list_raw.filter(
          (row) => row.status == filterstring
        );
      } else {
        taskListFiltered = this.task_list_raw;
      }

      taskListFiltered = taskListFiltered.map(function (currentObject) {
        currentObject["runtime"] =
          Math.ceil(currentObject.duration / 1000 / 60) + " min";
        return currentObject;
      });

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
        { text: "Runtime", value: "runtime" },
        { text: "Status", value: "status" },
      ];
    },
  },
};
</script>