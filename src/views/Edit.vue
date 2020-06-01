<template>
  <div>
    <div class="fix-top">
      <van-nav-bar
        fixed
        :title="headerTitle"
        :left-text="leftText"
        right-text="选择时间"
        @click-left="onClickLeft"
        @click-right="onClickRight"
      />
    </div>
    <div class="fix-middle" :style="styleObj">
      <van-form validate-first>
          <edit-module
            :info="editValue"
            :list="list"
            :columnsMap="columnsMap"
            :colIndex="0"
          />
      </van-form>
      <van-cell :title="item.txt" :value="item.cost" :label="item.message" value-class="van-cell-value" v-for="(item,index) in list" :key="index"/>
    </div>
      
    <van-popup v-model="show" position="bottom" :style="{ height: '30%' }">
      <div class="timepick">
        <van-datetime-picker
          safe-area-inset-bottom
          v-model="currentDate"
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
  TabbarItem,
  Picker,
  Field,
  Form,
  Cell
} from "vant";
import EditModule from "../components/EditModule.vue";
import obj from "../tools.js";
export default {
  data() {
    return {
      columns: [],
      columnsMap: [],
      showKeyboard: false,
      showPicker: false,
      pattern: /\d+(\.\d+|)/,
      value1: 0,
      tips: "",
      list: [],
      editValue: {
        txt: "",
        cost: 0,
        message: ""
      },
      typeVal: "",
      totalData: [],
      show: false,
      currentDate: new Date(),
      totalCost: 0,
      leftText: "保存"
    };
  },
  computed: {
    titleDate() {
      return this.$store.state.titleDate;
    },
    headerTitle() {
      return this.titleDate + " ¥" + this.thisTotal;
    },
    db() {
      return this.$store.state.db;
    },
    styleObj() {
      return `height: ${window.outerHeight - 96}px`;
    },
    thisTotal() {
      if (this.db[this.titleDate] && this.db[this.titleDate].totalCost) {
        return this.db[this.titleDate].totalCost;
      } else {
        return 0;
      }
    }
  },
  watch: {
    titleDate(val) {
      this.filterData();
      this.$router.push({
        query: { date: val }
      });
      return val;
    },
    db: {
      deep: true,
      handler: function(e) {
        e[this.titleDate].totalCost = this.list.reduce(
          (a, b) => a + parseFloat(b.cost),
          0
        );
        console.log("change", e);
        return e;
      }
    }
  },
  components: {
    vanrCol: Col,
    vanRow: Row,
    vanNavBar: NavBar,
    vanDatetimePicker: DatetimePicker,
    vanPopup: Popup,
    vanTabbar: Tabbar,
    vanTabbarItem: TabbarItem,
    vanField: Field,
    vanForm: Form,
    EditModule
  },

  methods: {
    onInput() {},
    onDelete() {},
    onConfirm(e) {
      this.typeVal = e;
      console.log(e);
    },
    filterData() {
      console.log("切换日期到", this.titleDate);
      if (this.db[this.titleDate]) {
        this.list = this.db[this.titleDate].thisdayCost;
      } else {
        Toast(`${this.titleDate}无数据`);
        this.$set(this.db, this.titleDate, {
          totalCost: 0,
          thisdayCost: []
        });
        this.list = this.db[this.titleDate].thisdayCost;
      }
      if (this.list.length === 0) {
        this.list.push({
          txt: "",
          cost: 0,
          message: ""
        });
      }
    },
    filter(type, options) {
      if (type === "day") {
        return options.filter(option => option % 1 === 0);
      }
      return options;
    },
    onClickLeft() {
      if (this.editValue.cost) {
        Toast("保存成功");
        this.list.push(this.editValue)
        this.saveData()
      }
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
    },
    showPopup() {
      this.show = true;
    },
    saveData() {
      const _arr = Object.keys(this.db);
      const obj = {};
      for (let i = _arr.length - 1; i >= 0; i--) {
        if (this.db[_arr[i]].totalCost === 0) {
          console.log("删除某日空数据", _arr[i]);
        } else {
          let _temp = [];
          this.db[_arr[i]].thisdayCost.forEach(e => {
            if (e.cost !== 0) {
              _temp.push(e);
            }
          });
          obj[_arr[i]] = {
            totalCost: this.db[_arr[i]].totalCost,
            thisdayCost: _temp
          };
        }
        console.log(obj);
      }

      lsg("db", obj);
    },
  },
  mounted() {
    console.log(2314, this.$route.query);
    if (this.$route.query.date) {
      this.$store.commit("changeDate", this.$route.query.date);
    }
    this.currentDate = new Date(this.titleDate);
    this.filterData();
    this.totalData = lsg("info") || [];
    console.log(this.totalData);
    this.columnsMap = Object.fromEntries(
      obj.statusMap.map(e => {
        return [e.type, e.subType.map(el => el.txt)];
      })
    );
  }
};
</script>
<style scoped>
.van-cell-value span{
  font-size: 18px;
  color: #000;
}
</style>