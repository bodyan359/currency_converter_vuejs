<template>
  <div id="app">
    <h1 class="title">Currency Converter</h1>
    <div class="container">
      <input
        class="currency-input"
        v-model="amountToConvert"
        @keypress="isNumber"
        @input="onInputDebounced"
        placeholder="Enter Amount to Convert"
      />
      <select class="select base" v-model="base">
        <option v-for="(item, index) in rates" :key="index">
          {{ item }}
        </option>
      </select>
      >
      {{ convertedAmount }}
      <select class="select currencyTo" v-model="currencyTo">
        <option v-for="(item, index) in rates" :key="index">
          {{ item }}
        </option>
      </select>
      <div class="result">
        {{ amountToConvert }} {{ base }}

        <button class="changeCurrencyButton" v-on:click="changeCurency">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="currentColor"
            height="16px"
            width="16px"
            viewBox="0 0 24 24"
            class="sc-AxhCb CnJdC"
          >
            <path
              d="M6 16H20M20 16L17 19M20 16L17 13"
              stroke="currentColor"
              stroke-width="2"
              stroke-miterlimit="10"
              stroke-linecap="round"
              stroke-linejoin="round"
            ></path>
            <path
              d="M18 8H4M4 8L7 11M4 8L7 5"
              stroke="currentColor"
              stroke-width="2"
              stroke-miterlimit="10"
              stroke-linecap="round"
              stroke-linejoin="round"
            ></path>
          </svg>
        </button>
        {{ currencyTo }}
      </div>
    </div>
    <h3 class="history">Currency Converter History</h3>
    <div
      class="result-history"
      v-for="(convertion, index) in latestConvertionsReverse"
      :key="index"
    >
      ({{ convertion.date }}): {{ convertion.amountToConvert }}
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
      amountToConvert: 10,
      convertedAmount: this.onInputDebounced,
      onInputDebounced: debounce(this.onInput, 1000),
    };
  },
  methods: {
    changeCurency: function() {
      let tmp, tmp2;
      tmp = this.base;
      tmp2 = this.currencyTo;
      this.base = tmp2;
      this.currencyTo = tmp;
    },
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

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #000000;
  margin-top: 60px;
  font-size: 24px;
}

.container {
  -ms-flex: 0 0 66.666667%;
  flex: 0 0 66.666667%;
  max-width: 66.666667%;
  border-radius: 30px;
  text-align: center;
  background: rgb(248, 248, 248);
  margin: auto;
}
@media (max-width: 900px) {
  .container {
    margin: 0px;
    max-width: 100%;
  }
}
.currency-input {
  height: 50px;
  font-size: 100%;
  font-weight: bold;
  border: 0;
  // padding: 7px 15px;
  border: 1px solid #ccc;
  position: relative;
  margin: 5px 10px;
}
.result {
  text-align: center;
  font-weight: bold;
  margin-top: 15px;
}
h1,
h3 {
  margin-left: 1.3em;
}
.result-history {
  margin-top: 15px;
  margin-left: 1.3em;
  border-bottom: 1px solid #2c3e50;
}
select {
  height: 50px;
  font-size: 100%;
  font-weight: bold;
  cursor: pointer;
  border-radius: 0;
  background: transparent;
  border: none;
  border-bottom: 2px solid #2b2424;
  color: rgb(0, 0, 0);
  appearance: none;
  padding: 10px;
  -webkit-appearance: none;
  -moz-appearance: none;
}
/* For IE <= 11 */
select::-ms-expand {
  display: none;
}
.changeCurrencyButton {
  transition: background 0.2s ease-in-out 0s;
  border-radius: 8px;
  font-weight: normal;
  display: inline-block;
  border: 1px solid transparent;
  margin: 0px;
  text-align: center;
  white-space: nowrap;
  cursor: pointer;
  padding: 0.375rem 0.75rem;
  font-size: 14px;
  line-height: 1.5;
  background: rgb(16, 112, 224);
  color: rgb(255, 255, 255);
  text-decoration: none !important;
  outline: none !important;
}
.changeCurrencyButton:hover {
  background: rgba(16, 112, 224, 0.85);
  color: rgb(255, 255, 255);
}
</style>
