<template>
  <div id="gradient" class="flex flex-col items-center h-screen justify-center">
    <div  class="flex flex-col items-start">
   <h3><td>股票代號：</td>
   <td><input v-on:keyup.enter="lastpricefunction" v-model="stockno" />{{ cname }}</td></h3>
      <h3><td>目標價：</td> <td><input v-model="targetp" /></td></h3>
      <h3><td>手續費折扣：</td> <td><input v-model="feediscount" /></td></h3>

      <table class="text-xl mt-2">
        <tr>
          <th>買價</th>
          <th>張數</th>
        </tr>
        <tr v-for="buy in buys" :key="buy.id">
          <td><input v-model="buy.b" /></td>
          <td><input v-model="buy.v" /></td>
        </tr>
      </table>

      <h3 class="mt-2">買入: {{ totalpaid }}</h3>
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
  </div>
</template>

<script>
// eslint-disable-next-line no-array-constructor
const colors = new Array(
  [62, 35, 255],
  [60, 255, 60],
  [255, 35, 98],
  [45, 175, 230],
  [255, 0, 255],
  [255, 128, 0],
);

let step = 0;

const colorIndices = [0, 1, 2, 3];

// transition speed
const gradientSpeed = 0.002;

function updateGradient() {
  // eslint-disable-next-line no-undef
  if ($ === undefined) return;
  // eslint-disable-next-line camelcase
  const c0_0 = colors[colorIndices[0]];
  // eslint-disable-next-line camelcase
  const c0_1 = colors[colorIndices[1]];
  // eslint-disable-next-line camelcase
  const c1_0 = colors[colorIndices[2]];
  // eslint-disable-next-line camelcase
  const c1_1 = colors[colorIndices[3]];

  const istep = 1 - step;
  const r1 = Math.round(istep * c0_0[0] + step * c0_1[0]);
  const g1 = Math.round(istep * c0_0[1] + step * c0_1[1]);
  const b1 = Math.round(istep * c0_0[2] + step * c0_1[2]);
  const color1 = `rgb(${r1},${g1},${b1})`;

  const r2 = Math.round(istep * c1_0[0] + step * c1_1[0]);
  const g2 = Math.round(istep * c1_0[1] + step * c1_1[1]);
  const b2 = Math.round(istep * c1_0[2] + step * c1_1[2]);
  const color2 = `rgb(${r2},${g2},${b2})`;

  // eslint-disable-next-line no-undef
  $('#gradient')
    .css({
      background:
        `-webkit-gradient(linear, left top, right top, from(${color1}),to(${color2}))`,
    })
    .css({ background: `-moz-linear-gradient(left, ${color1} 0%, ${color2} 100%)` });

  step += gradientSpeed;
  if (step >= 1) {
    step %= 1;
    // eslint-disable-next-line prefer-destructuring
    colorIndices[0] = colorIndices[1];
    // eslint-disable-next-line prefer-destructuring
    colorIndices[2] = colorIndices[3];

    // pick two new target color indices
    // do not pick the same as the current one
    // eslint-disable-next-line max-len
    colorIndices[1] = (colorIndices[1] + Math.floor(1 + Math.random() * (colors.length - 1))) % colors.length;
    // eslint-disable-next-line max-len
    colorIndices[3] = (colorIndices[3] + Math.floor(1 + Math.random() * (colors.length - 1))) % colors.length;
  }
}

setInterval(updateGradient, 10);

export default {
  name: 'main',
  data() {
    return {
      stockno: '1713',
      buys: [
        { id: 0, b: 17.7, v: 5 },
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
        if (buy.v !== '' || buy.b !== '') {
          const fee = buy.b * buy.v * 0.001425 * this.feediscount <= 20
            ? 20
            : buy.b * buy.v * 0.001425 * this.feediscount;
          tot += fee;
        }
      });
      return tot;
    },
    totalsellfee() {
      let tot = 0;
      this.buys.forEach((buy) => {
        if (buy.v !== '' || buy.b !== '') {
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

      // "error 404 d0527705"
    },
  },
};
</script>

<style lang="scss">
body {
  background-color: #000000;
  padding: 0px;
  margin: 0px;
}

#gradient {
  width: 100%;
  height: 800px;
  padding: 0px;
  margin: 0px;
}

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
