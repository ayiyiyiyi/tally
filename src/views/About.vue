<template>
  <div>
    <van-field label="Version" value="Beta0.1" readonly />
    <van-field label="Author" value="Mervin & ayi" readonly />
    <van-field label="Storage" :value="limit" readonly />
    <van-divider dashed>分割</van-divider>

    <van-field label="Import" v-model="val" />
    <van-button type="primary" size="large" @click="importData">Import Data</van-button>
    <van-divider dashed>分割</van-divider>
    <van-button type="primary" size="large" @click="showData">Checkout Data</van-button>
    <van-field v-model="infoDb" rows="1" label="数据" type="textarea" readonly />
  </div>
</template>

<script>
import { Toast } from "vant";
export default {
  name: "",
  data() {
    return {
      infoDb: "",
      val: ""
    };
  },
  computed: {
    limit() {
      if (!window.localStorage) {
        console.log("浏览器不支持localStorage");
      }
      var size = 0;
      for (let item in window.localStorage) {
        if (window.localStorage.hasOwnProperty(item)) {
          size += window.localStorage.getItem(item).length;
        }
      }

      return (size / 1024).toFixed(2) + "KB";
    },
    titleDate() {
      return this.$store.state.titleDate;
    },
    db() {
      return this.$store.state.db;
    }
  },
  methods: {
    showData() {
      this.infoDb = JSON.stringify(lsg("db"));
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
    importData() {
      console.log(this.val);
      if (this.val) {
        try {
          const obj = JSON.parse(this.val);
          if (typeof obj != "object") {
            throw new Error("转话错误");
          }
          lsg("db", obj);
        } catch {
          Toast("处理失败");
        }
      }
    }
  }
};
</script>