<template>
  <div id="app">
    <div class="wrapper">
      <h1>Currency Converter</h1>
      <div class="container">
        <div class="container-inside">
          <div class="currency-from">
            <div>
              <input
                class="currency-input"
                v-model="amountToConvert"
                @keypress="isNumber"
                @input="onInputDebounced"
                placeholder="Enter amount"
              />
            </div>
            <div>
              <select class="select base" v-model="base">
                <option v-for="(item, index) in rates" :key="index">
                  {{ item }}
                </option>
              </select>
            </div>
          </div>
          <div class="flex-space-between">
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
          </div>
          <div class="currency-to">
            <div>
              <input
                disabled
                class="currency-input"
                v-model="convertedAmount"
                @keypress="isNumber"
                placeholder="Result"
              />
            </div>
            <div>
              <select class="select currencyTo" v-model="currencyTo">
                <option v-for="(item, index) in rates" :key="index">
                  {{ item }}
                </option>
              </select>
            </div>
          </div>
        </div>
      </div>
      <div class="history">
        <h2>Currency Converter History</h2>
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
    changeCurency: function() {
      let tmp, tmp2, tmp3, tmp4;
      tmp = this.base;
      tmp2 = this.currencyTo;
      tmp3 = this.amountToConvert;
      tmp4 = this.convertedAmount;
      this.currencyTo = tmp;
      this.base = tmp2;
      this.convertedAmount = tmp3;
      this.amountToConvert = tmp4;
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

.history {
  text-align: center;
}

.container {
  background: aliceblue;
  border-radius: 30px;
  max-width: 1300px;
  margin: auto;
}

.container-inside {
  padding: 15px;
  margin: auto;
  display: flex;
  max-width: 800px;
}

.currency-input {
  height: 30px;
  max-width: 80%;
  font-size: 24px;
  font-weight: bold;
  margin: 5px 10px;
  text-align: center;
  border-radius: 17px;
}
@media (max-width: 378px) {
  .currency-input {
    max-width: 150px;
    font-size: 14px;
  }
}
.block {
  display: block;
}

.currency-from,
.currency-to {
  background-color: white;
  padding: 5px;
  border-radius: 15px;
  width: 300px;
}

.currency-from div,
.currency-to div {
  text-align: center;
}
.result {
  text-align: center;
  font-weight: bold;
  margin-top: 15px;
}
h1 {
  margin-top: 0;
}
h2 {
  margin-top: 85px;
  margin-bottom: 0;
}
.flex-space-between {
  margin: auto;
}
.result-history {
  border-bottom: 1px solid #2c3e50;
  max-width: 800px;
  margin: auto;
  padding-top: 20px;
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
  text-align: center;
  white-space: nowrap;
  cursor: pointer;
  padding-top: 5px;
  font-size: 14px;
  line-height: 1.5;
  background: rgb(16, 112, 224);
  color: rgb(255, 255, 255);
  height: 30px;
}
.changeCurrencyButton:hover {
  background: rgba(16, 112, 224, 0.85);
  color: rgb(255, 255, 255);
}
.wrapper {
  padding: 20px;
}
@media (max-width: 740px) {
  .container-inside {
    display: block;
    text-align: center;
  }

  .flex-space-between {
    margin: 20px 0;
  }
  .currency-from,
  .currency-to {
    width: unset;
  }
}
</style>
