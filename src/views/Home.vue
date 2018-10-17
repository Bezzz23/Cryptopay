<template>
  <div class="hello">
    <Header />
    <div class="exchange">
      <div class="exchange__header">Course</div>
      <div class="exchange__block">
        <div class="exchange__image"></div>
        <div class="exchange__text">1 {{crypt.currency}} = {{currencyPrice}} {{fiat.currency}}</div>
      </div>
    </div>
    <div class="currencyInputsBlock">
      <InputCurrency
        v-bind:options="cryptOptions"
        v-bind:text="crypt.text"
        v-bind:currency="crypt.currency"
        v-on:price-change="cryptChanged($event)"
        v-on:currency-change="cryptCurrencyChanged($event)"
      />
      <div class="currencyInputsBlock__switch">
        <img src="../assets/switch.svg" />
      </div>
      <InputCurrency
        v-bind:options="fiatOptions"
        v-bind:text="fiat.text"
        v-bind:currency="fiat.currency"
        v-on:price-change="fiatChanged($event)"
        v-on:currency-change="fiatCurrencyChanged($event)"
      />
    </div>
    <div class="chart-wrap">
    <apexcharts height="600" type="candlestick" :options="chartOptions" :series="series"></apexcharts>
    </div>
  </div>
</template>

<script>
import InputCurrency from '@/components/InputCurrency.vue'
import Header from '@/components/Header.vue'
import VueApexCharts from 'vue-apexcharts'
import axios from 'axios'

export default {
  name: 'HelloWorld',
  beforeMount: function () {
    this.getChart(this.crypt.currency, this.fiat.currency)
    this.cryptChanged(this.crypt)
  },
  data: function () {
    return {
      text: 'Test',
      currencyPrice: '',
      crypt: {
        text: '1',
        currency: 'BTC'
      },
      fiat: {
        text: '0',
        currency: 'USD'
      },
      cryptOptions: [
        { text: 'BTC', value: 'BTC' },
        { text: 'LTC', value: 'LTC' }
      ],
      fiatOptions: [
        { text: 'USD', value: 'USD' },
        { text: 'RUB', value: 'RUB' }
      ],
      chartOptions: {
        chart: {
          id: 'vuechart-example',
          height: 350,
        },
        title: {
          text: 'BTC/USD',
          align: 'left'
        },
        xaxis: {
          type: 'datetime',
          tooltip: {
              enabled: false
          }
        },
        candlestick: {
          colors: {
            upward: '#3C90EB',
            downward: '#DF7D46'
          }
        },
        yaxis: {
          tooltip: {
              enabled: false
          }
        }
      },
      series: null
    }
  },
  methods: {
    async cryptChanged (currency) {
      this.crypt = currency
      const priceData = await this.getPriceForCoin(currency.currency, this.fiat.currency)
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = currency.text * price + ''
    },
    async fiatChanged (currency) {
      this.fiat = currency
      const priceData = await this.getPriceForCoin(currency.currency, this.crypt.currency)
      let price = priceData.data[this.crypt.currency]
      this.crypt.text = currency.text * price + ''
    },
    async getChart (crypt, fiat) {
      const chartData = await axios.get(`https://min-api.cryptocompare.com/data/histoday?fsym=${crypt}&tsym=${fiat}&limit=150`)
      let data = chartData.data.Data.map((item) => {
        return [item.time, [item.open, item.high, item.low, item.close]]
      })
      this.series = [{data}]
    },
    async cryptCurrencyChanged (currency) {
      this.crypt.currency = currency.currency
      this.getChart(this.crypt.currency, this.fiat.currency)
      const priceData = await this.getPriceForCoin(this.crypt.currency, this.fiat.currency)
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = this.crypt.text * price + ''
    },
    async fiatCurrencyChanged (currency) {
      this.fiat.currency = currency.currency
      this.getChart(this.crypt.currency, this.fiat.currency)
      const priceData = await this.getPriceForCoin(this.crypt.currency, this.fiat.currency)
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = this.crypt.text * price + ''
    },
    getPriceForCoin (currency, otherCurrency) {
      return axios.get(`https://min-api.cryptocompare.com/data/price?fsym=${currency}&tsyms=${otherCurrency}`)
    }
  },
  components: {
    InputCurrency,
    apexcharts: VueApexCharts,
    Header
  }
}
</script>

<style scoped lang="scss">
.chart-wrap {
  padding-left: 50px;
  padding-right: 50px;
}
.exchange {
  margin-top: 40px;
  margin-bottom: 30px;
  &__header, &__text {
    font-weight: 500;
    font-size: 30px;
    color: #002A6F;
  }

  &__text {
    margin-top: 10px;
  }
}

.currencyInputsBlock {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  margin-bottom: 40px;

  &__switch {
    height: 40px;
    width: 40px;
    background: #1D65D2;
    margin-left: 30px;
    margin-right: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    transition: all 0.3s;
    cursor: pointer;

    &:hover {
      background: #3e82e8;
    }

    img {
      width: 15px;
    }
  }
}
</style>
