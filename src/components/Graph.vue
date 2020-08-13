<template>
  <div>
    <div class="chart-wrapper">
      <apexchart width="800" type="bar" :options="options" :series="series"></apexchart>
    </div>
    <!-- <hr />
    <div class="chart-wrapper">
      <apexchart width="800" type="line" :options="options" :series="series"></apexchart>
    </div>
    <hr />-->
  </div>
</template>

<script>
import apexchart from "vue-apexcharts";
export default {
  name: "Graph",
  data: () => ({
    options: {
      chart: {
        id: "vuechart-example"
      },
      stroke: {
        curve: "smooth"
      },
      tooltip: {
        enabled: true,
        theme: "dark",
        style: {
          fontSize: "12px",
          fontFamily: undefined,
          color: "#F00"
        }
      },
      xaxis: {
        categories: [
          "Jan",
          "Feb",
          "Mar",
          "Apr",
          "May",
          "Jun",
          "Jul",
          "Aug",
          "Sep",
          "Oct",
          "Nov",
          "Dec"
        ]
      },
      title: {
        text: "Monthly Stock Pricing",
        align: "center",
        style: {
          fontSize: "20px",
          color: "black"
        }
      },
      colors: ["#00897b"]
    },
    series: [
      {
        name: "series-1",
        data: [55, 62, 89, 66, 98, 72, 101, 75, 94, 120, 117, 139]
      }
    ]
  }),
  components: {
    apexchart
  }
};
</script>

<style scoped>
div.chart-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>