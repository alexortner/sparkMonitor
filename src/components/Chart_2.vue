<script>
import { Line } from "vue-chartjs";

export default {
  extends: Line,
  props: {
    label: {
      type: String,
    },
    chartData: {
      type: Array,
    },
    options: {
      type: Object,
    },
  },
  computed: {
    dataset(){
      let datasets = [
            {
              label: 'memoryUsed',
              borderColor: '#003f5c',
              fill: false,
              data: this.chartData.map((d) => d.memoryUsed)
            }, {
              label: 'maxMemory',
              fill: false,
              borderColor: '#2f4b7c',
              data: this.chartData.map((d) => d.maxMemory)
            },{
              label: 'JVMHeapMemory',
              borderColor: '#665191',
              fill: false,
              data: this.chartData.map((d) => d.JVMHeapMemory)
            },{
              label: 'OnHeapExecutionMemory',
              borderColor: '#a05195',
              fill: false,
              data: this.chartData.map((d) => d.OnHeapExecutionMemory)
            },{
              label: 'ProcessTreeJVMRSSMemory',
              borderColor: '#d45087',
              fill: false,
              data: this.chartData.map((d) => d.ProcessTreeJVMRSSMemory)
            },{
              label: 'ProcessTreeJVMVMemory',
              borderColor: '#f95d6a',
              fill: false,
              data: this.chartData.map((d) => d.ProcessTreeJVMVMemory)
            },{
              label: 'OnHeapUnifiedMemory',
              borderColor: '#ff7c43',
              fill: false,
              data: this.chartData.map((d) => d.OnHeapUnifiedMemory)
            }
          ]
        return datasets
        }
    }
    dataset(){
      const memoryUsed = this.chartData.map((d) => d.memoryUsed);
      console.log(this.chartData)
      let result = [
          {
            labels: this.label,
            data: memoryUsed,
          },
        ]
      return result
    },
    labels(){
       labels: this.memory_timeseries.map((d) => d.time),
      const dates = this.chartData.map((d) => d.time);
      return dates
    }

  },
  mounted() {
    const dates = this.chartData.map((d) => d.time);
    const memoryUsed = this.chartData.map((d) => d.memoryUsed);
    console.log(this.chartData)
    this.renderChart(
      {
        labels: dates,
        dataset: [
          {
            labels: this.label,
            data: memoryUsed,
          },
        ],
      },
      this.options
    );
  },
};
</script>