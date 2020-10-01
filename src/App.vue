<template>
  <div class="flex flex-col items-center">
    <h1 class="my-10">當沖好幫手</h1>
    <div class="flex justify-center">

      <div class="flex flex-col h-screen mx-4 text-center" id="priceRange">
        <table class="text-xl bg-gray-200 border-2">
          <tr>
            <th>成交價</th>
            <th>損益</th>
          </tr>
          <tr v-for="p in pRange" :key="p.id" :class="p.price==avg ? 'bg-red-200' : 'bg-blue-100'">
            <td>{{totalpaid === 0 ? '-' : p.price}}</td>
            <td>{{totalpaid === 0 ? '-' : Math.round(p.nett)}}</td>
          </tr>
        </table>
      </div>

      <div class="flex flex-col items-center h-screen mx-4">
        <h2>
          股票代號：<input
            v-on:keyup.enter="lastPriceFunction"
            v-model="stockno"
          />{{ cname }}
        </h2>
        <h2>目標價：<input v-model.number="targetp" /></h2>
        <h2>手續費折扣：<input v-model.number="feediscount" /></h2>

        <table class="text-xl">
          <tr>
            <th>買價</th>
            <th>張數</th>
          </tr>
          <tr v-for="buy in buys" :key="buy.id">
            <td><input v-model.number="buy.b" /></td>
            <td><input v-model.number="buy.v" /></td>
          </tr>
        </table>
      </div>

        <div class="flex flex-col h-screen mx-4">
          <h2>買入: {{ totalpaid }}</h2>
          <h2>現在賣: {{ targetp * totalV }}</h2>
          <br />
          <h2>買手續費： {{ totalpaid === 0 ? '' : Math.round(totalBuyFee) }}</h2>
          <h2>賣手續費： {{ totalpaid === 0 ? '' : Math.round(totalSellFee) }}</h2>
          <h2>稅0.003/2： {{ totalpaid === 0 ? '' : Math.round(tax) }}</h2>
          <br />
          <h2>原收入： {{ totalpaid === 0 ? '' : income }}</h2>
          <h2>稅+手續費： {{ totalpaid === 0 ? '' : Math.round(allfees) }}</h2>
          <h2 class="bg-red-600">總收入： {{ Math.round(nett) }}</h2>
          <br />
          <h2>最後收盤: {{ lprice }}</h2>
          <h2>每升一格多少元: {{ totalpaid === 0 ? '' : tick }}</h2>
          <h2>成本價: {{ avg === 0 ? '' : avg.toFixed(2) }}</h2>
        </div>
      </div>
    </div>
</template>

<script>
export default {
  name: 'main',
  data() {
    return {
      stockno: '',
      buys: [
        { b: null, v: null },
        { b: null, v: null },
        { b: null, v: null },
        { b: null, v: null },
        // { b: 10, v: 1000 },
        // { b: 9, v: 1000 },
        // { b: 8, v: 1000 },
        // { b: 12, v: 1000 },
        // { b: 8.5, v: 1000 },
      ],
      lprice: null,
      targetp: null,
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
    totalBuyFee() {
      let tot = 0;
      this.buys.forEach((buy) => {
        if (buy.v !== null || buy.b !== null) {
          const fee = buy.b * buy.v * 0.001425 * this.feediscount <= 20
            ? 20
            : buy.b * buy.v * 0.001425 * this.feediscount;
          tot += fee;
        }
      });
      return tot;
    },
    totalSellFee() {
      let tot = 0;
      this.buys.forEach((buy) => {
        if (buy.v !== null || buy.b !== null) {
          const fee = this.targetp * buy.v * 0.001425 * this.feediscount <= 20
            ? 20
            : this.targetp * buy.v * 0.001425 * this.feediscount;
          tot += fee;
        }
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
      return (this.totalpaid - this.targetp * this.totalV) * -1;
    },
    allfees() {
      return this.totalBuyFee + this.totalSellFee + this.tax;
    },
    nett() {
      return this.income - this.allfees;
    },
    tick() {
      if (this.targetp < 10) return 0.01;
      if (this.targetp < 50) return 0.05;
      if (this.targetp < 100) return 0.1;
      if (this.targetp < 500) return 0.5;
      if (this.targetp < 1000) {
        return 1;
      } return 5;
    },
    pRange() {
      const array = [];
      let price = parseFloat(this.targetp) + (10 * this.tick);
      for (let index = 0; index < 19; index += 1) {
        array.push({ price: `${price.toFixed(2)}`, nett: `${price * this.totalV - this.totalpaid - this.totalBuyFee - this.calculateFee(price) - this.tax}` });
        price -= this.tick;
      }
      return array;
    },
    avg() {
      if (this.totalpaid !== 0) {
        const avg = (this.totalpaid + this.totalBuyFee) / this.totalV;
        const newAvg = avg % this.tick === 0 ? avg : avg + this.tick - (avg % this.tick);
        return newAvg;
      }
      return 0;
      //  ? null : newAvg.toFixed(2);
    },
    computedPrice() {
      return typeof this.lprice === 'undefined' ? this.avg : this.lprice;
    },
  },
  mounted() {
    this.lastPriceFunction();
    this.targetp = this.computedPrice;
  },
  methods: {
    lastPriceFunction() {
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
    calculateFee(sellPrice) {
      const fee = sellPrice * this.totalV * 0.001425 * this.feediscount <= 20
        ? 20
        : sellPrice * this.totalV * 0.001425 * this.feediscount;
      return fee;
    },
  },
};
</script>

<style lang="scss">
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  background: linear-gradient(90deg, #e3ffe7 0%, #d9e7ff 100%);
}
th,
td {
  width: 10vw;
}
#priceRange th {
  @apply py-1;
}
#priceRange td,th {
  width: 10vw;
  @apply px-3 border;
}

input {
  background-color: lightgreen;
  width: 10vw;
}

h1 {
  font-size: 72px;
  background: -webkit-linear-gradient(#eee, #333);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

h2 {
  font-size: 36px;
}
</style>
