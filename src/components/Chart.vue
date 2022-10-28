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
let initData = []
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
          data: initData
        },{
          type: 'line',
          name: 'Max',
          data: []
        },{
          type: 'line',
          name: 'Mid',
          data: []
        },{
          type: 'line',
          name: 'Min',
          data: []
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
            if (key === this.symbol && value.candle) {
              let data = [], maxData = [], midData = [], minData = [];
              Object.values(value.candle).map(el => {
                data.push([el.time * 1000, el.open, el.high, el.low, el.close])
                maxData.push([el.time * 1000, value.calc.max])
                midData.push([el.time * 1000, value.calc.mid])
                minData.push([el.time * 1000, value.calc.min])
              })
              this.chartOptions.series[0].data = data.slice().reverse()
              this.chartOptions.series[1].data = maxData.slice().reverse()
              this.chartOptions.series[2].data = midData.slice().reverse()
              this.chartOptions.series[3].data = minData.slice().reverse()
            }
          }
        })
        // console.log(this.priceData)
      });
    },
    selectSymbol (ev) {
      this.symbol = ev.target.value
      console.log(this.symbol, this.priceData.get(this.symbol).candle[1])
      if (this.priceData.get(this.symbol) && this.priceData.get(this.symbol)?.candle) {
        let data = [], maxData = [], midData = [], minData = []
        Object.values(this.priceData.get(this.symbol).candle).map(el => {
          data.push([el.time * 1000, el.open, el.high, el.low, el.close])
          maxData.push([el.time * 1000, this.priceData.get(this.symbol).calc.max])
          midData.push([el.time * 1000, this.priceData.get(this.symbol).calc.mid])
          minData.push([el.time * 1000, this.priceData.get(this.symbol).calc.min])
        })
        this.chartOptions.series[0].data = data.slice().reverse()
        this.chartOptions.series[1].data = maxData.slice().reverse()
        this.chartOptions.series[2].data = midData.slice().reverse()
        this.chartOptions.series[3].data = minData.slice().reverse()
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
