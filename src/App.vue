<template>
  <div id="app">
    <div class="col">
      <h1 class="title">Currency Converter</h1>
    </div>
    <div>
      <div class="">
        <input
          id="currencyInput"
          class="currency-input"
          v-model="amountToConvert"
          @keypress="isNumber"
          @input="onInput"
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
    <div>
      {{ amountToConvert }} {{ base }}
      =
      {{ convertedAmount }}
      {{ currencyTo }}
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  components: {},
  computed: {
    rates: {
      get: function() {
        return this.rates_internal ? Object.keys(this.rates_internal) : [];
      },
      set: function(value) {
        this.rates_internal = value;
      },
    },
  },
  data() {
    return {
      last_convertions: [],
      rates_internal: null,
      base: "USD",
      currencyTo: "PLN",
      amountToConvert: "",
      convertedAmount: "",
    };
  },
  methods: {
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
        });
    },
  },
  mounted() {
    axios
      .get("https://api.exchangeratesapi.io/latest")
      .then((response) => {
        this.rates_internal = response.data.rates ? response.data.rates : [];
      })
      .catch((error) => {
        console.error(error);
      });
  },
  watch: {
    base: "onInput",
    currencyTo: "onInput",
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
