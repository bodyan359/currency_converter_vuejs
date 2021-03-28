<template>
  <div id="app">
    <div class="col">
      <h1 class="title">Currency Converter</h1>
    </div>
    <div>
      <div class="">
        <input
          class="currency-input"
          v-model="amountToConvert"
          @keypress="isNumber"
          @input="onInputDebounced"
          placeholder="Enter Amount to Convert"
        />
      </div>
      <select v-model="base">
        <option v-for="(item, index) in rates" :key="index">
          {{ item }}
        </option>
      </select>
      to
      <select v-model="currencyTo">
        <option v-for="(item, index) in rates" :key="index">
          {{ item }}
        </option>
      </select>
    </div>
    <div class="">
      {{ amountToConvert }} {{ base }}
      =
      {{ convertedAmount }}
      {{ currencyTo }}
    </div>
    <div v-for="(convertion, index) in latestConvertionsReverse" :key="index">
      ({{ convertion.date }}) : {{ convertion.amountToConvert }}
      {{ convertion.base }} = {{ convertion.convertedAmount }}
      {{ convertion.currencyTo }}
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { debounce, dateFormat } from "./utils";

export default {
  name: "App",
  components: {},
  computed: {
    latestConvertionsReverse: function() {
      return this.latestConvertions.slice(0).reverse();
    },
  },
  data() {
    return {
      latestConvertions: [],
      rates: [],
      base: "USD",
      currencyTo: "PLN",
      amountToConvert: "",
      convertedAmount: "",
      onInputDebounced: debounce(this.onInput, 1000),
    };
  },
  methods: {
    addLatestConvertions: function() {
      this.latestConvertions.push({
        date: dateFormat(new Date()),
        amountToConvert: this.amountToConvert,
        base: this.base,
        currencyTo: this.currencyTo,
        convertedAmount: this.convertedAmount,
      });
    },
    isNumber: function(evt) {
      evt = evt ? evt : window.event;
      let charCode = evt.which ? evt.which : evt.keyCode;
      if (
        charCode > 31 &&
        (charCode < 48 || charCode > 57) &&
        charCode !== 46
      ) {
        evt.preventDefault();
        return false;
      } else {
        return true;
      }
    },
    onInput: function() {
      axios
        .get(
          `https://api.exchangeratesapi.io/latest?base=${this.base}&symbols=${this.currencyTo}`
        )
        .then((response) => {
          let exchangeRate = Object.values(response.data.rates)[0];
          this.convertedAmount = (exchangeRate * this.amountToConvert).toFixed(
            2
          );
          this.addLatestConvertions();
        });
    },
  },
  mounted() {
    axios
      .get("https://api.exchangeratesapi.io/latest")
      .then((response) => {
        this.rates = response.data.rates
          ? Object.keys(response.data.rates)
          : [];
      })
      .catch((error) => {
        console.error(error);
      });
  },
  watch: {
    base: function() {
      this.onInputDebounced();
    },
    currencyTo: function() {
      this.onInputDebounced();
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
