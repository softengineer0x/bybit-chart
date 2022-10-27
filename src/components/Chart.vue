<template>
  <div class="hello">
    <div class="chart">
      <select v-model="symbol" @change="selectSymbol($event)">
        <option value="">Select Pair</option>
        <option v-for="option in options" :key="option.value" :value="option.value">
          {{option.label}}
        </option>
      </select>
    <highcharts :constructor-type="'stockChart'" :options="chartOptions"></highcharts>
    </div>
  </div>
</template>

<script>
import {Chart} from 'highcharts-vue'
import { io } from 'socket.io-client';
let initData = [[1603805400000,115.49,117.28,114.54,116.6],[1603891800000,115.05,115.43,111.1,111.2],[1603978200000,112.37,116.93,112.2,115.32],[1604064600000,111.06,111.99,107.72,108.86],[1604327400000,109.11,110.68,107.32,108.77],[1604413800000,109.66,111.49,108.73,110.44],[1604500200000,114.14,115.59,112.35,114.95]]
export default {
  name: 'ChartComponent',
  components: Chart,
  created() {
    this.getData();
  },
  beforeUnmount() {
    if (this.socket) {
      this.socket.disconnect();
    }
  },
  data() {
    return {
      socket: null,
      priceData: new Map(),
      options: [],
      symbols: [],
      symbol: '',
      chartOptions: {
        rangeSelector: {
          selected: 1
        },

        title: {
          text: 'Bybit Price'
        },

        series: [{
          type: 'candlestick',
          name: 'Bybit Price',
          data: initData,
        }]
      }
    }
  },
  methods: {
    async getData() {
      this.socket = io('http://localhost:5000');
      this.socket.on('updateData', (data) => {
        console.log(data);
        // console.log(data.symbols)
        if (data.symbols) {
          this.symbols = data.symbols;
          this.options = data.symbols.map(el => {
            return {label: el, value: el}
          })
        }
        Object.entries(data).forEach(([key, value]) => {
          if (this.symbols.includes(key)) {
            this.priceData.set(key, value);
          }
        })
        // console.log(this.priceData)
      });
    },
    selectSymbol (ev) {
      this.symbol = ev.target.value
      console.log(this.symbol, this.priceData.get(this.symbol).candle[1])
      if (this.priceData.get(this.symbol) && this.priceData.get(this.symbol)?.candle) {
        let data = []
        Object.values(this.priceData.get(this.symbol).candle).map(el => {
          console.log(el.time, el.open, el.high, el.low, el.close)
          data.push([el.time * 1000, el.open, el.high, el.low, el.close])
        })
        this.chartOptions.series[0].data = data.slice().reverse()
        this.chartOptions.title.text = `${this.symbol} Price`
        this.chartOptions.series[0].name = `${this.symbol} Price`
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.chart {
  width: 70%;
  margin-left: 15%;
  margin-right: 15%;
  text-align: right;
}
</style>
