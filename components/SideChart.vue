<template>
  <div class="legend pb-3" :class="{ rightz: !left, left: left }">
    <div class="container-fluid h-100 px-0">
      <div class="card h-100 bg-light" style="border-radius: 20px">
        <div>
          <h5 class="w-100 text-center">
            {{ wilayah }}
          </h5>
        </div>
        <div>
          <canvas id="myChart"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { party } from "@/assets/party";
import Chart from "chart.js/auto";
// const chartColors = {
//   red: 'rgb(255, 99, 132)',
//   orange: 'rgb(255, 159, 64)',
//   yellow: 'rgb(255, 205, 86)',
//   green: 'rgb(75, 192, 192)',
//   blue: 'rgb(54, 162, 235)',
//   purple: 'rgb(153, 102, 255)',
//   grey: 'rgb(201, 203, 207)',
// }
export default {
  props: {
    data: {},
    wilayah: String,
    dataChart: "[]",
    isAceh: {
      type: Boolean,
      default: false,
    },
    left: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      handler: false,
      myChart:null,
      images: [
        "https://i.stack.imgur.com/2RAv2.png",
        "https://i.stack.imgur.com/Tq5DA.png",
        "https://i.stack.imgur.com/3KRtW.png",
        "https://i.stack.imgur.com/iLyVi.png",
      ],
      barChartData: {
        labels: [],
        datasets: [
          {
            label: "Perloehan Suara",
            backgroundColor: [],
            data: [],
          },
        ],
      },
      dataChartParsed: [],
      barChartOptions: {
        responsive: true,
        title: {
          display: true,
          text: "Perolehan Suara",
        },
        legend: {
          display: false,
        },
        scales: {
          xAxes: [
            {
              ticks: {
                beginAtZero: true,
              },
            },
          ],
        },
      },
      dataReady: false,
    };
  },
  methods: {
    getData() {
      this.dataReady = false;
      this.barChartData.datasets[0].backgroundColor = [];
      this.barChartData.labels = [];
      this.barChartData.datasets[0].data = [];
      Object.keys(party).forEach((key) => {
        this.barChartData.labels.push(party[key].nama);
        this.barChartData.datasets[0].backgroundColor.push(party[key].warna);
        this.barChartData.datasets[0].data = JSON.parse(this.dataChart);
      });
      this.barChartData.datasets[0].backgroundColor.splice(14, 4);
      this.barChartData.labels.splice(14, 4);
      this.barChartData.datasets[0].data.splice(14, 4);
      this.dataReady = true;
    },
  },
  mounted() {
    this.getData();
    this.$nextTick(function () {
      const ctx = document.getElementById("myChart").getContext("2d");
      this.myChart = new Chart(ctx, {
        type: "bar",
        data: this.barChartData,
        options: {
          indexAxis: "y",
          // Elements options apply to all of the options unless overridden in a dataset
          // In this case, we are setting the border of each horizontal bar to be 2px wide
          elements: {
            bar: {
              borderWidth: 2,
            },
          },
          responsive: true,
          plugins: {
            legend: {
              position: "right",
            },
            title: {
              display: true,
              text: "Chart.js Horizontal Bar Chart",
            },
          },
        },
      });
    });
  },
  watch: {
    wilayah() {
      this.getData();
      this.handler = !this.handler;
      this.dataChartParsed = JSON.parse(this.dataChart);
      this.myChart?.update();
    },
  },
};
</script>
<style>
.chartStyle {
  height: "1500px";
}
</style>
