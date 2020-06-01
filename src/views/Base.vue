<template>
  <div>
    <div class="fix-top">
      <van-nav-bar
        fixed
        :title="titleDate"
        :left-text="leftText"
        right-text="选择时间"
        @click-left="onClickLeft"
        @click-right="onClickRight"
      />
    </div>
    <div class="fix-middle" :style="styleObj">
      <van-row>
        <van-col span="24">
          <!-- <pie-chart :pieChartData="chartData" /> -->
          <bar-chart :barChartData.sync="todayChartData" :day="titleDate" />
          <!-- <pie-chart :chartData.sync="todayChartData" /> -->
          <line-chart :lineChartData.sync="yesterWeekChartData" :day="titleDate"/>
        </van-col>
      </van-row>
    </div>
    <van-popup v-model="show" position="bottom" :style="{ height: '30%' }">
      <div class="timepick">
        <van-datetime-picker
          safe-area-inset-bottom
          v-model="currentDate"
          cancel-button-text="回到今天"
          type="date"
          title="选择年月日"
          :filter="filter"
          @change="changePickTime"
          @confirm="confrimPickTime"
          @cancel="cancelPickTime"
        />
      </div>
    </van-popup>
  </div>
</template>

<script>
import {
  Col,
  Row,
  NavBar,
  Toast,
  DatetimePicker,
  Popup,
  Tabbar,
  TabbarItem
} from "vant";
import PieChart from "../components/PieChart.vue";
import BarChart from "../components/BarChart.vue";
import LineChart from "../components/LineChart.vue";
import obj from "../tools.js";
export default {
  name: "Base",
  components: {
    vanrCol: Col,
    vanRow: Row,
    vanNavBar: NavBar,
    vanDatetimePicker: DatetimePicker,
    vanPopup: Popup,
    vanTabbar: Tabbar,
    vanTabbarItem: TabbarItem,
    BarChart,
    PieChart,
    LineChart
  },
  data() {
    return {
      todayChartData: {},
      yesterChartData: {},
      yesterWeekChartData: [],
      show: false,
      currentDate: new Date(),
      leftText: ""
    };
  },
  computed: {
    titleDate() {
      return this.$store.state.titleDate;
    },
    yestertitleDate() {
      return this.pastDate(this.titleDate);
    },
    db() {
      return lsg("db") || [];
    },
    styleObj() {
      return `height: ${window.outerHeight - 96}px`;
    }
  },
  watch: {
    titleDate(val) {
      // this.titleDate = val;
      this.filterData();
      return val;
    }
  },
  methods: {
    // tool
    pastDate(date, n = 1) {
      return formatDate(
        new Date(new Date(date).getTime() - n * 24 * 60 * 60 * 1000)
      );
    },
    // bar 数据
    filterBarDataByDate(date) {
      // 处理今日和昨日
      if (this.db[date]) {
        const _temp = this.db[date].thisdayCost;
        const _tempMap = {};
        for (let i = _temp.length - 1; i >= 0; i--) {
          if (_tempMap[_temp[i].txt]) {
            _tempMap[_temp[i].txt] += parseFloat(_temp[i].cost);
          } else {
            _tempMap[_temp[i].txt] = parseFloat(_temp[i].cost);
          }
        }
        return _tempMap;
      } else {
        Toast(`${date}无数据`);
        return undefined;
      }
    },
    // line 数据
    filterLineDataByDate(date) {
      const _obj = [];
      for (let i = 0; i < 7; i++) {
        let pas = this.pastDate(this.titleDate, i);
        console.log(pas);
        _obj[pas] = this.db[pas] || { totalCost: 0 };
      }
      console.log(45, _obj);
      return _obj;
    },
    filterData() {
      // barchart数据
      this.todayChartData = this.filterBarDataByDate(this.titleDate);
      this.yesterChartData = this.filterBarDataByDate(this.yestertitleDate);
      //
      this.yesterWeekChartData = this.filterLineDataByDate(this.titleDate);
    },
    filter(type, options) {
      if (type === "day") {
        return options.filter(option => option % 1 === 0);
      }
      return options;
    },
    onClickLeft() {
      Toast("待开发");
      if (this.$route.name == "Edit") {
        this.$store.commit("changeDB");
      }
      // this.$router.go(-1);
    },
    onClickRight() {
      this.show = true;
    },
    changePickTime() {},
    confrimPickTime(e) {
      this.$store.commit("changeDate", formatDate(e));
      this.currentDate = e;
      this.show = false;
    },
    cancelPickTime() {
      this.show = false;
      const date = formatDate(new Date());
      this.$store.commit("changeDate", date);
    },
    showPopup() {
      this.show = true;
    },
    
  },
  created() {
    this.currentDate = new Date(this.titleDate);
    this.filterData();
  }
};
</script>