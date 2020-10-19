<template>
  <div class="flex flex-col items-center">
    <div id="title" class="my-4 text-5xl md:mb-6 md:mt-2 md:text-6xl">當沖好幫手</div>
    <div class="flex flex-col md:flex-row justify-center">

      <div class="flex flex-col mx-4 text-center" id="priceRange">
        <table class="text-xl bg-gray-200 border-2">
          <tr>
            <th>成交價</th>
            <th>損益</th>
          </tr>
          <tr
            v-for="p in pRange"
            :key="p.id"
            :class="p.price == avg ? 'bg-yellow-200'
            : p.price == targetp ? 'bg-red-200' : 'bg-blue-100'"
          >
            <td>{{ totalPaid === 0 ? '-' : p.price }}</td>
            <td>{{ totalPaid === 0 ? '-' : Math.round(p.nett) }}</td>
          </tr>
        </table>
      </div>

      <div class="flex flex-col items-center mt-6 mx-46">
        <h2 class="self-start">
          股票代號：<input
            v-on:keyup.enter="lastPriceFunction"
            v-model="stockno"
            inputmode="decimal"
            class="w-1/3 right-0 md:w-32"
          />{{ cname }}
        </h2>
        <h2 class="self-start">
          目標價：<input v-model.number="targetp" inputmode="decimal" class="w-1/3 md:w-32" />
        </h2>
        <h2 class="self-start">手續費折扣：
          <input v-model.number="feeDiscount" inputmode="decimal" class="w-1/4 md:w-32" />
        </h2>

        <table class="mt-4 text-3xl">
          <tr>
            <th class="w-1/2">買價</th>
            <th class="w-1/2">張數</th>
          </tr>
          <tr v-for="buy in buys" :key="buy.id">
            <td>
              <input v-model.number="buy.b" inputmode="decimal" class="block mx-auto md:mx-0" />
            </td>
            <td>
              <input v-model.number="buy.v" inputmode="decimal" class="block mx-auto md:mx-0"/>
            </td>
          </tr>
        </table>
      </div>

        <div class="flex flex-col h-screen mx-4 mt-6">
          <h3>買入: {{ totalPaid }}</h3>
          <h3>現在賣: {{ totalReceive }}</h3>
          <br />
        <h3>買手續費： {{ totalPaid === 0 ? '' : totalBuyFee }}</h3>
        <h3>
          賣手續費： {{ totalPaid === 0 ? '' : totalSellFee }}
        </h3>
        <h3>稅0.003/2： {{ totalPaid === 0 ? '' : tax }}</h3>
          <br />
          <h2>原收入： {{ totalPaid === 0 ? '' : income }}</h2>
          <h2>稅+手續費： {{ totalPaid === 0 ? '' : Math.round(allfees) }}</h2>
          <h2 class="bg-red-600">總收入： {{ Math.round(nett) }}</h2>
          <br />
          <h3>最後收盤: {{ lprice }}</h3>
          <h3>每升一格多少元: {{ totalPaid === 0 ? '' : tick }}</h3>
          <h2>成本價: {{ avg === 0 ? '' : avg.toFixed(2) }}</h2>
        </div>
      </div>
    </div>
</template>

<script>
export default {
  name: 'homepage',
  data() {
    return {
      stockno: '',
      buys: [
        { b: null, v: null },
        { b: null, v: null },
        { b: null, v: null },
        { b: null, v: null },
        { b: null, v: null },
        { b: null, v: null },
        // { b: 39.5, v: 1 },
        // { b: 39.55, v: 1 },
        // { b: 39.4, v: 1 },
        // { b: 12, v: 1000 },
        // { b: 8.5, v: 1000 },
      ],
      lprice: null,
      targetp: null,
      feeDiscount: 0.3,
      cname: '',
    };
  },
  computed: {
    totalPaid() {
      let tot = 0;
      this.buys.forEach((buy) => {
        tot += buy.b * buy.v * 1000;
      });
      return Math.round(tot);
    },
    totalV() {
      let tot = 0;
      this.buys.forEach((buy) => {
        tot += buy.v * 1000;
      });
      return tot;
    },
    totalReceive() {
      return Math.round(this.targetp * this.totalV);
    },
    totalBuyFee() {
      let tot = 0;
      this.buys.forEach((buy) => {
        if (buy.v !== null || buy.b !== null) {
          const fee = buy.b * buy.v * 1000 * 0.001425 * this.feeDiscount <= 20
            ? 20
            : buy.b * buy.v * 1000 * 0.001425 * this.feeDiscount;
          tot += fee;
        }
      });
      return Math.floor(tot);
    },
    totalSellFee() {
      // let tot = 0;
      // this.buys.forEach((buy) => {
      // if (buy.v !== null || buy.b !== null) {
      //   const fee = this.targetp * buy.v * 0.001425 * this.feeDiscount <= 20
      //     ? 20
      //     : this.targetp * buy.v * 0.001425 * this.feeDiscount;
      //   tot += fee;
      // }
      // });
      const tot = this.totalReceive * 0.001425 * this.feeDiscount <= 20
        ? 20
        : this.totalReceive * 0.001425 * this.feeDiscount;

      return Math.floor(tot);
    },
    tax() {
      let tot = 0;
      this.buys.forEach((buy) => {
        const fee = (this.targetp * buy.v * 1000 * 0.003) / 2;
        tot += fee;
      });
      return Math.floor(tot);
    },
    income() {
      return (this.totalPaid - this.totalReceive) * -1;
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
      }
      return 5;
    },
    pRange() {
      const array = [];
      let price = parseFloat(this.targetp) + 8 * this.tick;
      for (let index = 0; index < 17; index += 1) {
        array.push({
          price: `${price.toFixed(2)}`,
          nett: `${
            price * this.totalV
            - this.totalPaid
            - this.totalBuyFee
            - this.calculateFee(price)
            - this.tax
          }`,
        });
        price -= this.tick;
      }
      return array;
    },
    avg() { // 成本價買賣家與買價相同，稅依照買價計算
      if (this.totalPaid !== 0) {
        const tempTax = (this.totalPaid * 0.003) / 2;
        const avg = (this.totalPaid + this.totalBuyFee * 2 + tempTax) / this.totalV;
        const newAvg = avg % this.tick === 0 ? avg : avg + this.tick - (avg % this.tick);
        return newAvg;
      }
      return 0;
      //  ? null : newAvg.toFixed(2);
    },
    // computedPrice() {
    //   return typeof this.lprice === 'undefined' ? this.avg : this.lprice;
    // },
  },
  // mounted() {
  //   this.targetp = this.computedPrice;
  // },
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
      const fee = sellPrice * this.totalV * 0.001425 * this.feeDiscount <= 20
        ? 20
        : sellPrice * this.totalV * 0.001425 * this.feeDiscount;
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
  // width: 10vw;
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
  width: 25vw;
}
@media (min-width: 768px) {
  input {width: 10vw;}
}

div#title {
  background: -webkit-linear-gradient(#eee, #333);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

h2 {
  font-size: 36px;
}
h3 {
  font-size: 24px;
}
</style>
