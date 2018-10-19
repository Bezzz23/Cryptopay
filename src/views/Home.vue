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
      <Graph v-bind:stock="stock" v-bind:graph="series"/>
    </div>
    
  </div>
</template>

<script>
import InputCurrency from '@/components/InputCurrency.vue'
import Header from '@/components/Header.vue'
import axios from 'axios'
import m from 'moment'
import Graph from '@/components/Graph.vue'

let api = 'https://min-api.cryptocompare.com'

export default {
  name: 'Home',
  beforeMount: function () {
    this.getChart(this.crypt.currency, this.fiat.currency)
    this.cryptChanged(this.crypt)
  },
  data: function () {
    return {
      stock: {},
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
          zoom: {
              enabled: false,
          }
        },
        xaxis: {
          type: 'datetime',
        }
      },
      series: []
    }
  },
  methods: {
    async cryptChanged (currency) {
      this.crypt = currency
      let priceData
      try {
        priceData = await this.getPriceForCoin(currency.currency, this.fiat.currency)
      } catch (err) {
        this.handleErrors(err)
      }
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = (currency.text * price).toFixed(4) + ''
    },
    async fiatChanged (currency) {
      this.fiat = currency
      let priceData
      try {
        priceData = await this.getPriceForCoin(currency.currency, this.crypt.currency)
      } catch (err) {
        this.handleErrors(err)
      }
      let price = priceData.data[this.crypt.currency]
      this.crypt.text = (currency.text * price).toFixed(4) + ''
    },
    async getChart (crypt, fiat) {
      const chartData = await axios.get(`https://min-api.cryptocompare.com/data/histoday?fsym=${crypt}&tsym=${fiat}&limit=150`)
      let data = chartData.data.Data.map((item) => ({
         date: m.unix(item.time).format('YYYY-MM-DD'),
        close: item.close,
        low: item.low,
        high: item.high,
        open: item.open,
        volume: item.volumefrom
      }))
      this.series = data
    },
    async cryptCurrencyChanged (currency) {
      if (currency.currency === this.fiat.currency) {
        return alert('Please select other coin')
      }
      this.crypt.currency = currency.currency
      this.getChart(this.crypt.currency, this.fiat.currency)
      let priceData
      try {
        priceData = await this.getPriceForCoin(this.crypt.currency, this.fiat.currency)
      } catch (err) {
        this.handleErrors(err)
      }
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = (this.crypt.text * price).toFixed(4) + ''
    },
    async fiatCurrencyChanged (currency) {
      if (currency.currency === this.crypt.currency) {
        return alert('Please select other coin')
      }
      this.fiat.currency = currency.currency
      this.getChart(this.crypt.currency, this.fiat.currency)
      let priceData
      try {
        priceData = await this.getPriceForCoin(this.crypt.currency, this.fiat.currency)
      } catch (err) {
        this.handleErrors(err)
      }
      let price = priceData.data[this.fiat.currency]
      this.currencyPrice = price
      this.fiat.text = (this.crypt.text * price).toFixed(4) + ''
    },
    getPriceForCoin (currency, otherCurrency) {
      return axios.get(`${api}/data/price?fsym=${currency}&tsyms=${otherCurrency}`)
    },
    async switchCurrency () {
      let fiatText = this.fiat.text
      this.fiat.text = this.crypt.text
      this.crypt.text = fiatText

      let fiatCurrency = this.fiat.currency
      this.fiat.currency = this.crypt.currency
      this.crypt.currency = fiatCurrency

      this.cryptChanged(this.crypt)
      this.getChart(this.crypt.currency, this.fiat.currency)
    },
    handleErrors(err) {
      if (err.status === 404) {
        alert("The specified symbol could not be found...");
      } else {
        alert("There was an error retrieving the data...");
      }
    },
  },
  components: {
    InputCurrency,
    Header,
    Graph
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
