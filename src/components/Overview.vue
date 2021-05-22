<template>
  <v-container>
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
    <pre></pre>
    <!-- <v-row>
      <v-col>
         <v-list-item
          v-for="(item, i) in executors"
          :key="i"
        >
          <v-list-item-content>
            <v-list-item-title v-text="item.id"></v-list-item-title>
            Pod Memory: {{Math.ceil((item.peakMemoryMetrics.JVMHeapMemory+item.peakMemoryMetrics.JVMOffHeapMemory)/1000000000)}} GB<br>
            Executor Memory: {{Math.ceil(item.memoryUsed/1000000000)}} GB /  {{Math.ceil(item.maxMemory/1000000000)}} GB<br>
            
          </v-list-item-content>
        </v-list-item>
         {{this.executors[2]}}
      </v-col>
    </v-row>-->
    <v-row>
      <!-- Left Card -->
      <v-col cols="6" class="mb-4">
        <v-card class="mx-auto" max-width="600">
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                Executor Details
              </v-list-item-title>
              <v-list-item-subtitle>
                <b>Number Executors:</b>
                {{ executors_number }}</v-list-item-subtitle
              >
              <v-list-item-subtitle
                ><b>Pod Memory:</b>
                {{ environment["spark.executor.memory"] }}</v-list-item-subtitle
              >
              <v-list-item-subtitle
                ><b>Executor Memory:</b>
                {{
                  Math.ceil(
                    executors[executor_random_id].maxMemory / 1000000000
                  )
                }}G</v-list-item-subtitle
              >
            </v-list-item-content>
          </v-list-item>
          <v-divider></v-divider>

          <v-card-subtitle class="pb-0">
            Memory (Executor Id:{{ this.executors[executor_random_id].id }})
          </v-card-subtitle>
          <v-card-text class="text--primary pb-0">
            <v-progress-linear
              v-model="memory_used_percent"
              height="25"
              color="amber"
            >
              <strong
                >{{
                  Math.ceil(
                    executors[executor_random_id].memoryUsed / 1000000000
                  )
                }}G/
                {{
                  Math.ceil(
                    executors[executor_random_id].maxMemory / 1000000000
                  )
                }}G
              </strong>
            </v-progress-linear>
          </v-card-text>

          <v-card-subtitle class="pb-0 pt-2">
            Cores (Executor Id:{{
              this.executors[executor_random_id].id
            }})</v-card-subtitle
          >
          <v-card-text class="text--primary">
            <v-progress-linear
              v-model="cores_used_percent"
              height="25"
              color="red"
            >
              <strong>
                {{ executors[executor_random_id].activeTasks }}/
                {{ executors[executor_random_id].totalCores }}
              </strong>
            </v-progress-linear>
          </v-card-text>
        </v-card>
      </v-col>
      <!-- Right Card -->
      <v-col cols="6" class="mb-4">
        <v-card class="mx-auto" max-width="600">
          <v-list-item two-line>
            <v-list-item-content>
              <v-list-item-title class="headline">
                Job Details
              </v-list-item-title>
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
                >{{
                  active_job.numCompletedStages + active_job.numSkippedStages
                }}/{{ active_job.stageIds.length }} ({{
                  active_job.numActiveStages
                }}
                running, {{ active_job.numFailedStages }} failed)</strong
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
        <h1>Drill down into Details</h1>
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
                    <v-tooltip
                      left
                      v-if="item.status === 'FAILED'"
                      max-width="300"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-chip
                          v-bind="attrs"
                          v-on="on"
                          :color="getColor(item.status)"
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
                      :color="getColor(item.status)"
                      dark
                      small
                    >
                      {{ item.status }}
                    </v-chip>
                  </template>
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

          <v-expansion-panel>
            <v-expansion-panel-header>
              <span style="width: 10%">Executor Memory Monitor </span>
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <div><pre>
                {{ chartExecutorMemory2["1"].chartData}}
                </pre>
                <pre>
                {{chartExecutorMemory}}
                </pre>
              </div>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-col>

      <v-col class="mb-5" cols="12">
        Single CHART TESTING {{startTime}}
        <chart :chart-data="chartExecutorMemory" :options="chartOptions"></chart>
      </v-col>
    </v-row>
    <v-row>
    <v-col>
   
    </v-col>
    </v-row>
 


    
  </v-container>
</template>


<script>
import axios from "axios";
import Chart from "./Chart.js"

export default {
  name: "Overview",
  components: {
    Chart
  },
  data: () => ({
    spark_api: "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications",
    spark_api_current_app: "",
    rancher_api:"https://svr-rancher.local.parcit/v3",
    application_id: null,
    application_name: "",
    application_starttime: null,
    application_runtime: "",
    application_json: null,
    environment: {},
    job_stages: [],
    executors: [],
    jobs: [],
    tasklist: [],
    search: "",
    filter: "",
    memory_timeseries: [],
    ExecutorMemory: [],
    chartOptions: {
      responsive: true,
      maintainAspectRatio: false,
    },
    chartExecutorMemory: null,
    chartExecutorMemory2:{},
    startTime: 0,
    InitialRun:false,
    executor1: null,
  }),
  created() {
    this.apiCallWrapper();
    setInterval(this.apiCallWrapper, 6000);
  },
  mounted () {
    this.createChartData()
  },

  methods: {

     async apiCallWrapper() {
      try {
        //await this.getRancherAPI();
        this.setStartTime();
        await this.getApplicationInfo();
        await this.getEnvironmentDetails();
        await this.getJopDetails();
        await this.getStageDetails();
        await this.getCurrentTaskList();
        await this.getExecutorDetails();
        await this.append_memory_timeseries();
        await this.createChartData();
        await this.createMemoryChartData();
      } catch (e) {
        this.$emit("heartbeat", "api error");
        console.log(e);
      }
    },

    async createMemoryChartData(){
      let now=new Date().getTime();
      let time=(now-this.startTime)/1000

      //run once to create object structure
      if(this.InitialRun==false){
        for (var j = 0; j < 2; j++) {
          this.chartExecutorMemory2[this.executors[j].id]={
            name : this.executors[j].id,
            chartData : {
              labels: [],
              datasets: [
                {
                  label: 'memoryUsed',
                  data: []
                }, 
                {
                  label: 'maxMemory',
                  data: []
                }
                ]
            }
        }
      }
      this.InitialRun=true
      }

      for (var i = 0; i < 2; i++) {
          this.chartExecutorMemory2[this.executors[i].id].chartData.labels.push(time)
          this.chartExecutorMemory2[this.executors[i].id].chartData.datasets[0].data.push(Math.ceil(this.executors[i].memoryUsed/1000000000))
          this.chartExecutorMemory2[this.executors[i].id].chartData.datasets[1].data.push(Math.ceil(this.executors[i].maxMemory/1000000000))
          }
      //this.test=this.chartExecutorMemory2
      //console.log("FICKER",this.chartExecutorMemory2)
      this.executor1=this.chartExecutorMemory2["1"].chartData
    },
    setStartTime(){
      if(this.startTime==0){
        this.startTime=new Date().getTime();
      }
    },
    async createChartData () {
      //console.log(this.memory_timeseries.map((d) => d.memoryUsed))
        this.chartExecutorMemory = {
          labels: this.memory_timeseries.map((d) => d.time),
          datasets: [
            {
              label: 'memoryUsed',
              borderColor: '#003f5c',
              pointRadius: 0,
              fill: false,
              data: this.memory_timeseries.map((d) => d.memoryUsed)
            }, {
              label: 'maxMemory',
              fill: false,
              borderColor: '#2f4b7c',
              pointRadius: 0,
              data: this.memory_timeseries.map((d) => d.maxMemory)
            },{
              label: 'JVMHeapMemory',
              borderColor: '#665191',
              fill: false,
              pointRadius: 0,
              data: this.memory_timeseries.map((d) => d.JVMHeapMemory)
            },{
              label: 'OnHeapExecutionMemory',
              borderColor: '#a05195',
              pointRadius: 0,
              fill: false,
              data: this.memory_timeseries.map((d) => d.OnHeapExecutionMemory)
            },{
              label: 'ProcessTreeJVMRSSMemory',
              borderColor: '#d45087',
              pointRadius: 0,
              fill: false,
              data: this.memory_timeseries.map((d) => d.ProcessTreeJVMRSSMemory)
            },{
              label: 'ProcessTreeJVMVMemory',
              borderColor: '#f95d6a',
              pointRadius: 0,
              fill: false,
              data: this.memory_timeseries.map((d) => d.ProcessTreeJVMVMemory)
            },{
              label: 'OnHeapUnifiedMemory',
              borderColor: '#ff7c43',
              pointRadius: 0,
              fill: false,
              data: this.memory_timeseries.map((d) => d.OnHeapUnifiedMemory)
            }
          ]
        }
    },
    async getApplicationInfo() {
      const response = await axios.get(this.spark_api, {
        headers: {
          "Access-Control-Allow-Credentials": "true",
          "Content-Type": "application/json",
        },
      });
      this.$emit("heartbeat", response.statusText);
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

    async getJopDetails() {
      const response = await axios.get(
        this.spark_api.concat("/", this.application_id, "/jobs"),
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

    async getEnvironmentDetails() {
      const response = await axios.get(
        this.spark_api.concat("/", this.application_id, "/environment"),
        {
          headers: {
            "Access-Control-Allow-Credentials": "true",
            "Content-Type": "application/json",
          },
        }
      );
      let environment_set = {};
      let sparkProperties = response.data.sparkProperties;
      for (var i = 0; i < sparkProperties.length; i++) {
        //total_tasks=total_tasks+this.job_stages[i].numTasks
        environment_set[sparkProperties[i][0]] = sparkProperties[i][1];
      }
      this.environment = environment_set;
    },

    async getStageDetails() {
      const response = await axios.get(
        this.spark_api.concat("/", this.application_id, "/stages"),
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

    async getExecutorDetails() {
      const response = await axios.get(
        this.spark_api.concat("/", this.application_id, "/executors"),
        {
          headers: {
            "Access-Control-Allow-Credentials": "true",
            "Content-Type": "application/json",
          },
        }
      );
      //console.log(response.data);
      this.executors = response.data;
      this.executors.sort((a, b) => (a.id > b.id ? 1 : -1));
    },

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
          "?length=400"
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
    async getRancherAPI() {
      //var basicAuth = 'Basic ' + btoa("token-6znph" + ':' + "pg7cfmd7st57ndq6q49mw5jnnrh4xbnchvcq8p4n2msbmwhckb94p6");
      var token = "alor:wbdSHr!!34"
      const response = await axios.get(this.rancher_api, {
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${token}`
        }
      });

      console.log("kubernetes: ", response);
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
    
    async append_memory_timeseries() {
      let date = new Date();
      let now = date.getTime();
      let executor_id=6 //this.executor_random_id
      
      let entry = {
        time: now,
        memoryUsed: Math.ceil(this.executors[executor_id].memoryUsed / 1000000000),
        maxMemory: Math.ceil(this.executors[executor_id].maxMemory / 1000000000),
        usedOnHeapStorageMemory: Math.ceil(this.executors[executor_id].memoryMetrics.usedOnHeapStorageMemory / 1000000000),
        usedOffHeapStorageMemory: Math.ceil(this.executors[executor_id].memoryMetrics.usedOffHeapStorageMemory / 1000000000),
        totalOnHeapStorageMemory: Math.ceil(this.executors[executor_id].memoryMetrics.totalOnHeapStorageMemory / 1000000000),
        totalOffHeapStorageMemory: Math.ceil(this.executors[executor_id].memoryMetrics.totalOffHeapStorageMemory / 1000000000),

        JVMHeapMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.JVMHeapMemory / 1000000000),
        JVMOffHeapMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.JVMOffHeapMemory / 1000000000),
        OnHeapExecutionMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.OnHeapExecutionMemory / 1000000000),
        OffHeapExecutionMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.OffHeapExecutionMemory / 1000000000),
        OnHeapStorageMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.OnHeapStorageMemory / 1000000000),

        OffHeapStorageMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.OffHeapStorageMemory / 1000000000),
        OnHeapUnifiedMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.OnHeapUnifiedMemory / 1000000000),
        OffHeapUnifiedMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.OffHeapUnifiedMemory / 1000000000),
        DirectPoolMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.DirectPoolMemory / 1000000000),
        MappedPoolMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.MappedPoolMemory / 1000000000),

        ProcessTreeJVMVMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.ProcessTreeJVMVMemory / 1000000000),
        ProcessTreeJVMRSSMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.ProcessTreeJVMRSSMemory / 1000000000),
        ProcessTreePythonVMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.ProcessTreePythonVMemory / 1000000000),
        ProcessTreePythonRSSMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.ProcessTreePythonRSSMemory / 1000000000),
        ProcessTreeOtherVMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.ProcessTreeOtherVMemory / 1000000000),

        ProcessTreeOtherRSSMemory: Math.ceil(this.executors[executor_id].peakMemoryMetrics.ProcessTreeOtherRSSMemory / 1000000000),
        MinorGCCount: Math.ceil(this.executors[executor_id].peakMemoryMetrics.MinorGCCount / 1000000000),
        MinorGCTime: Math.ceil(this.executors[executor_id].peakMemoryMetrics.MinorGCTime / 1000000000),
        MajorGCCount: Math.ceil(this.executors[executor_id].peakMemoryMetrics.MajorGCCount / 1000000000),
        MajorGCTime: Math.ceil(this.executors[executor_id].peakMemoryMetrics.MajorGCTime / 1000000000),
      };
      this.memory_timeseries.push(entry);
    },
  },

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
    executors_number() {
      // substract driver/master
      return this.executors.length - 1;
    },
    executor_random_id() {
      return Math.floor(Math.random() * Number(this.executors.length - 1));
    },
    job_number() {
      return this.jobs.length;
    },
    task_number() {
      return this.jobs[0].numTasks;
    },
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
    memory_used_percent() {
      return Math.ceil(
        (Number(
          this.executors[this.executor_random_id].memoryUsed / 1000000000
        ) /
          Number(
            this.executors[this.executor_random_id].maxMemory / 1000000000
          )) *
          100
      );
    },
    cores_used_percent() {
      return Math.ceil(
        (Number(this.executors[this.executor_random_id].activeTasks) /
          Number(this.executors[this.executor_random_id].totalCores)) *
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

      if (filterstring != "") {
        taskListFiltered = this.tasklist.filter(
          (row) => row.status == filterstring
        );
      } else {
        taskListFiltered = this.tasklist;
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
