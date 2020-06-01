<template>
  <div class>
    <div class="chart" ref="chart"></div>
  </div>
</template>

<script>
export default {
  name: "BarChart",
  data() {
    return {};
  },
  props: ["barChartData", "day"],
  computed: {
    titleDate() {
      return this.$store.state.titleDate;
    }
  },
  watch: {
    titleDate(val) {
      this.setOptions();
      return val;
    },
    barChartData: {
      deep: true,
      handler() {
        // this.setOptions();
      }
    }
  },
  mounted() {
    this.initCharts();
  },
  methods: {
    initCharts() {
      this.chart = $E.init(this.$refs.chart);
      this.setOptions();
    },
    handelrData(obj) {
      if (obj) {
        const _total = Object.values(obj).reduce((a, b) => a + b);
        const opt = {
          title: {
            text: `${this.day}消费`,
            show: true,
            // left: "20%",
            // textAlign: "center",
            // textVerticalAlign: "middle",
            subtext: `${_total}元`
          },
          color: ['#E5C1CD','#F3D8FC','#AAC9CE'],
          xAxis: {
            type: "category",
            data: Object.keys(obj),
            nameTextStyle: {
              color: '#AAC9CE'
            },
            axisTick: false
          },
          yAxis: {
            type: "value",
            "nameTextStyle": {
              color: '#AAC9CE'
            },
            axisTick: false,
            "splitLine": {
              "show": false
            }
          },
          series: [
            {
              title: { text: "今日消费" },
              label: {
                show: true,
                position: "top",
              },
              barMaxWidth: 10,
              data: Object.values(obj),
              type: "bar",
            }
          ]
        };
        return opt;
      } else {
        return {
          title: {
            text: `${this.day}消费`,
            show: true,
            subtext: `0元`
          },
          xAxis: {
            type: "category",
            data: ["居然没有花钱"]
          },
          yAxis: {
            type: "value"
          },
          series: [
            {
              title: { text: "今日消费" },
              label: {
                show: true,
                position: "top"
              },
              data: [0],
              type: "bar"
            }
          ]
        };
      }
    },
    setOptions() {
      const opt = this.handelrData(this.barChartData);
      console.log("aetaa", opt);
      this.chart.setOption(opt);
    }
  }
};
</script>

<style>
.chart {
  width: 100vw;
  height: 60vw;
}
</style>
