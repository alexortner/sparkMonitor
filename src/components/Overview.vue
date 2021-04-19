<template>
  <v-container>
    <v-row >
      <v-col cols="6">
         <v-card class="mx-auto" max-width="400">
            <v-list-item two-line>
              <v-list-item-content>
                <v-list-item-title class="headline">
                     Running: {{application_name}}
                </v-list-item-title>
                <v-list-item-subtitle>Id: {{application_id}}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>

            <v-list-item>
               <v-list-item-title>Runtime:</v-list-item-title>
             
             
              <v-list-item-subtitle >{{application_runtime}}</v-list-item-subtitle>
               <v-list-item-icon class="text-right">
                <v-icon>mdi-clock</v-icon>
              </v-list-item-icon>
            </v-list-item>

            <v-divider></v-divider>


              <v-list class="transparent">
      <v-list-item>
        <v-list-item-title>Jobs:</v-list-item-title>
        <v-list-item-subtitle >{{job_number}}</v-list-item-subtitle>
      </v-list-item>
      <v-list-item>
        <v-list-item-title>Stages:</v-list-item-title>
        <v-list-item-subtitle >{{stage_number}}</v-list-item-subtitle>
      </v-list-item>
      <v-list-item>
        <v-list-item-title>Tasks:</v-list-item-title>
        <v-list-item-subtitle >{{application_runtime}}</v-list-item-subtitle>
      </v-list-item>
    </v-list>

   
  </v-card>
       
      </v-col>

      <v-col class="mb-4">
      <v-btn
      depressed
      color="primary"
      @click="getJobDetails()"
    >
      Run Axios
    </v-btn>

     

      </v-col>

      <v-col
        class="mb-5"
        cols="12"
      >
      <!-- <div v-for="stage in job_stages" v-bind:key="stage.stageId">
     name: {{ stage.name }}<br>
     id: {{ stage.stageId }}<br>
     status: {{ stage.status }}<br>
     Total Tasks: {{ stage.numTasks }}<br>
     Aktive Tasks: {{ stage.numActiveTasks}}<br>
     CompleteTasks: {{ stage.numCompleteTasks }}<br>
     GC Time {{stage.jvmGcTime}}<br> 


    

     
      </div>-->


      </v-col>

      <v-col
        class="mb-5"
        cols="12"

      >
     JSON RAW <pre>{{active_stage}}</pre>

      </v-col>

    
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
  export default {
    name: 'Overview',

    data: () => ({
      application_id:null,
      application_name:"",
      application_starttime:null,
      application_runtime:"",
      application_json:null,
      job_stages:[],
      jobs:[]
    }),
    created () {
      this.getStageDetails()
      setInterval(this.getStageDetails,3000)
    },
    methods: {
      getStageDetails(){
        axios.get('http://spark-ui.k8s-prod1.local.parcit/api/v1/applications',
          {headers: {
                "Access-Control-Allow-Credentials":"true",
                "Content-Type": "application/json"
                }
          })
        .then(response => {
          let now = new Date()
          this.application_id=response.data[0].id
          this.application_name=response.data[0].name
          this.application_starttime=response.data[0].attempts[response.data[0].attempts.length - 1].startTime

          // calculate runtime
          let start=this.application_starttime
          let starttime = new Date(start.substr(0, 4), (Number(start.substr(5, 2))-1).toString(), start.substr(8, 2), (Number(start.substr(11, 2))+2).toString(), start.substr(14, 2), start.substr(17, 2));
          let difference=new Date(now-starttime)
          const seconds = (Math.floor((difference / 1000) % 60)).toString()
          const minutes = (Math.floor((difference / 1000 / 60) % 60)).toString()
          const hours = (Math.floor((difference  / 1000 / 3600 ) % 24)).toString()
          this.application_runtime=hours.concat(":",minutes,":",seconds)
          console.log("current runtime: ", this.application_runtime)
          
          axios.get('http://spark-ui.k8s-prod1.local.parcit/api/v1/applications/'.concat(response.data[0].id,'/stages'),
              {headers: {
                  "Access-Control-Allow-Credentials":"true",
                  "Content-Type": "application/json"
              }
        })
        .then(response => {
          this.job_stages=response.data
          this.job_stages.sort((a, b) => (a.stageID > b.stageID ? 1 : -1))

           axios.get('http://spark-ui.k8s-prod1.local.parcit/api/v1/applications/'.concat( this.application_id,'/jobs'),
              {headers: {
                  "Access-Control-Allow-Credentials":"true",
                  "Content-Type": "application/json"
              }
        }).then(response => {
          this.jobs=response.data
          this.jobs.sort((a, b) => (a.jobID > b.jobID ? 1 : -1))
        })
          
      })
   
        })
       
      }
    },
    computed: {
      stage_number(){
        return this.job_stages.length
      },
      job_number(){
        return this.jobs.length
      },
      active_job(){
        return this.jobs.filter(entry=>entry.status=="RUNNING")
      },
      active_stage(){
        return this.job_stages.filter(entry=>entry.status=="ACTIVE")
      }
    }
  }
</script>
