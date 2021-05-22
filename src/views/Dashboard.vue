<template>
  <v-container>
    <!-- Top Bar below Navigation -->
    <SummaryTop
      :application_runtime="application_runtime"
      :application_id="application_id"
      :spark_api_current_app="spark_api_current_app"
      :jobs="jobs" 
      :job_stages="job_stages"
    ></SummaryTop>

    <v-row>
      <!-- Left Card -->
      <v-col cols="6" class="mb-4">
        <SummaryExecutors
          :executors="executors"
          :pod_memory="environment['spark.executor.memory']"
        >
        </SummaryExecutors>
      </v-col>
      <!-- Right Card -->
      <v-col cols="6" class="mb-4">
        <SummaryJobs :jobs="jobs" :job_stages="job_stages"> </SummaryJobs>
      </v-col>
      <!-- Details in Expansion Panels-->
      <v-col class="mb-5" cols="12">
      <h1>Detail View</h1>
        <DetailsTasks 
          :spark_api="spark_api"
          :application_id="application_id"
          :job_stages="job_stages"
        >
        </DetailsTasks>
      </v-col>
    </v-row>
    <v-row>
      <v-col> </v-col>
    </v-row>
  </v-container>
</template>


<script>
import axios from "axios";
import SummaryTop from "../components/SummaryTop";
import SummaryExecutors from "../components/SummaryExecutors";
import SummaryJobs from "../components/SummaryJobs";
import DetailsTasks from "../components/DetailsTasks";

export default {
  name: "Overview",
  components: {
    SummaryTop,
    SummaryExecutors,
    SummaryJobs,
    DetailsTasks,
  },
  data: () => ({
    spark_api: "http://spark-ui.k8s-prod1.local.parcit/api/v1/applications",
    environment: {},
    job_stages: [],
    executors: [],
    jobs: [],
    tasklist: null,
    interval: null
  }),
  props: [
      'application_runtime',
      'application_id',
      'spark_api_current_app',
      'application_name'
      ],
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
        await this.getEnvironmentDetails();
        await this.getJopDetails();
        await this.getStageDetails();
        //await this.getCurrentTaskList();
        await this.getExecutorDetails();
      } catch (e) {
        this.$emit("heartbeat", "api error");
        console.log(e);
      }
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

  },

}
</script>
