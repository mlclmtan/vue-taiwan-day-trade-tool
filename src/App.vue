<template>
  <div id="app">
    <h3>
      股票代號：<input
        v-on:keyup.enter="lastpricefunction"
        v-model="stockno"
      />{{ cname }}
    </h3>
    <h3>目標價：<input v-model="targetp" /></h3>
    <h3>手續費折扣：<input v-model="feediscount" /></h3>

    <table class="text-xl">
      <tr>
        <th>買價</th>
        <th>張數</th>
      </tr>
      <tr v-for="buy in buys" :key="buy.id">
        <td><input v-model="buy.b" /></td>
        <td><input v-model="buy.v" /></td>
      </tr>
    </table>
    <h3>買入: {{ totalpaid }}</h3>
    <h3>現在賣: {{ targetp * totalV }}</h3>
    <br />
    <h3>買手續費： {{ totalbuyfee }}</h3>
    <h3>賣手續費： {{ totalsellfee }}</h3>
    <h3>稅0.003/2： {{ tax }}</h3>
    <br />
    <h3>原收入： {{ income }}</h3>
    <h3>稅+手續費： {{ allfees }}</h3>
    <h3 class="bg-red-600">總收入： {{ nett }}</h3>
    <br />
    <h3>最後收盤: {{ lprice }}</h3>
  </div>
</template>

<script>
export default {
  name: 'main',
  data() {
    return {
      stockno: '1713',
      buys: [
        { id: 0, b: 17.7, v: 5000 },
        { id: 0, b: '', v: '' },
        { id: 0, b: '', v: '' },
        { id: 0, b: '', v: '' },
        { id: 0, b: '', v: '' },
      ],
      lprice: '',
      targetp: '',
      feediscount: 0.3,
      cname: '',
    };
  },
  computed: {
    totalpaid() {
      let tot = 0;
      this.buys.forEach((buy) => {
        tot += buy.b * buy.v;
      });
      return tot;
    },
    totalV() {
      let tot = 0;
      this.buys.forEach((buy) => {
        tot += buy.v;
      });
      return tot;
    },
    totalbuyfee() {
      let tot = 0;
      this.buys.forEach((buy) => {
        const fee = buy.b * buy.v * 0.001425 * this.feediscount <= 20
          ? 20
          : buy.b * buy.v * 0.001425 * this.feediscount;
        tot += fee;
        console.log(fee);
      });
      return tot;
    },
    totalsellfee() {
      let tot = 0;
      this.buys.forEach((buy) => {
        const fee = this.targetp * buy.v * 0.001425 * this.feediscount <= 20
          ? 20
          : this.targetp * buy.v * 0.001425 * this.feediscount;
        tot += fee;
      });
      return tot;
    },
    tax() {
      let tot = 0;
      this.buys.forEach((buy) => {
        const fee = (this.targetp * buy.v * 0.003) / 2;
        tot += fee;
      });
      return tot;
    },
    income() {
      return this.totalpaid - this.targetp * this.totalV;
    },
    allfees() {
      return this.totalbuyfee + this.totalsellfee + this.tax;
    },
    nett() {
      return this.income - this.allfees;
    },
  },
  mounted() {
    this.lastpricefunction();
  },
  methods: {
    lastpricefunction() {
      const proxyurl = 'https://cors-anywhere.herokuapp.com/';
      const url = `https://mis.twse.com.tw/stock/api/getStockInfo.jsp?ex_ch=otc_${this.stockno}.tw&json=1&delay=0`;
      fetch(proxyurl + url)
        .then((response) => response.text())
        .then((contents) => {
          if (JSON.parse(contents).msgArray.length === 0) {
            this.other(
              `https://mis.twse.com.tw/stock/api/getStockInfo.jsp?ex_ch=tse_${this.stockno}.tw&json=1&delay=0`,
            );
          }
          this.lprice = JSON.parse(contents).msgArray[0].z;
          this.targetp = this.lprice;
          this.cname = JSON.parse(contents).msgArray[0].n;
        });
      // .catch(() =>
      //   console.log("Can’t access " + url + " response. Blocked by browser1?")
      // );
    },
    other(url) {
      const proxyurl = 'https://cors-anywhere.herokuapp.com/';
      fetch(proxyurl + url)
        .then((response) => response.text())
        .then((contents) => {
          // if(JSON.parse(contents).msgArray.length == 0) {
          //   console.log('Wrong stock')
          // }
          this.lprice = JSON.parse(contents).msgArray[0].z;
          this.targetp = this.lprice;
          this.cname = JSON.parse(contents).msgArray[0].n;
        });
      // .catch(() =>
      //   console.log("Can’t access " + url + " response. Blocked by browser1?")
      // );
    },
  },
};
</script>

<style lang="scss">
@tailwind base;

@tailwind components;

@tailwind utilities;
th,
td {
  width: 10vw;
}

input {
  background-color: lightgreen;
}
</style>
