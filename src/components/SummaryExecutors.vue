<template>
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
                {{ pod_memory }}</v-list-item-subtitle
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
</template>
<script>
export default {
  name: "SummaryExecutors",
  data: () => ({
  
  }),
  props: [
      'executors',
      'pod_memory'
      ],
  computed: {
    executors_number() {
      // substract driver/master
      return this.executors.length - 1;
    },
    executor_random_id() {
      return Math.floor(Math.random() * Number(this.executors.length - 1));
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
  }
}
</script>
