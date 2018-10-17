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
      <div class="currencyInputsBlock__switch" v-on:click="switchCurrency">
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
  name: 'Home',
  beforeMount: function () {
    this.getChart(this.crypt.currency, this.fiat.currency)
    this.cryptChanged(this.crypt)
  },
  data: function () {
    return {
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
        { text: 'LTC', value: 'LTC' },
        { text: 'USD', value: 'USD' },
        { text: 'RUB', value: 'RUB' }
      ],
      fiatOptions: [
        { text: 'BTC', value: 'BTC' },
        { text: 'LTC', value: 'LTC' },
        { text: 'USD', value: 'USD' },
        { text: 'RUB', value: 'RUB' }
      ],
      chartOptions: {
        chart: {
          id: 'vuechart-example',
          height: 350,
        },
        xaxis: {
          type: 'datetime',
        }
      },
      series: [{
        data: [0]
      }]
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
      if (currency.currency === this.fiat.currency) {
        return alert('Please select other coin')
      }
      this.crypt.currency = currency.currency
      this.getChart(this.crypt.currency, this.fiat.currency)
      const priceData = await this.getPriceForCoin(this.crypt.currency, this.fiat.currency)
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = this.crypt.text * price + ''
    },
    async fiatCurrencyChanged (currency) {
      if (currency.currency === this.crypt.currency) {
        return alert('Please select other coin')
      }
      this.fiat.currency = currency.currency
      this.getChart(this.crypt.currency, this.fiat.currency)
      const priceData = await this.getPriceForCoin(this.crypt.currency, this.fiat.currency)
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = this.crypt.text * price + ''
    },
    getPriceForCoin (currency, otherCurrency) {
      return axios.get(`https://min-api.cryptocompare.com/data/price?fsym=${currency}&tsyms=${otherCurrency}`)
    },
    async switchCurrency () {
      let fiatText = this.fiat.text
      this.fiat.text = this.crypt.text
      this.crypt.text = fiatText

      let fiatCurrency = this.fiat.currency
      this.fiat.currency = this.crypt.currency
      this.crypt.currency = fiatCurrency

      this.cryptChanged(this.crypt)
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
