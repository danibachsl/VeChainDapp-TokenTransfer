<template>
  <div id="app">
    <div>
      <b-card style="max-width: 30rem" :header="myaddress">
        <p>{{ myamount }} <span class="text-primary">VTHO</span></p>

        <b-form-group label="To Address:" label-for="toaddress">
          <b-form-input id="toaddress" v-model.trim="toaddress"></b-form-input>
        </b-form-group>

        <b-form-group label="Transfer Amount" label-for="toamount">
          <b-input-group append="VTHO">
            <b-form-input
              id="toamount"
              type="number"
              min="0"
              v-model.number="toamount"
            >
            </b-form-input>
          </b-input-group>
        </b-form-group>

        <b-button variant="primary" v-on:click="transfer"> Transfer </b-button>
      </b-card>
    </div>
  </div>
</template>

<script>
const operations = require("./operations.js");
const utils = require("./utils.js");
export default {
  data() {
    return {
      myaddress: "0x4d176ee98977B052cce2B6E3b7b192beb6198e8b",
      myamount: 0,
      contract: "0x0000000000000000000000000000456e65726779",
      toaddress: "",
      toamount: 0,
    };
  },
  beforeMount() {
    this.refreshTokenBalance();
  },
  methods: {
    refreshTokenBalance() {
      operations
        .getTokenBalance(this.contract, this.myaddress)
        .then((result) => {
          this.myamount = utils.evmToPrintable(
            result["decoded"]["balance"],
            this.decimals
          );
        })
        .then(() => {
          // eslint-disable-next-line
          const ticker = connex.thor.ticker();
          ticker.next().then(() => {
            this.refreshTokenBalance();
          });
        })
        // eslint-disable-next-line no-unused-vars
        .catch((e) => {
          setTimeout(() => {
            this.refreshTokenBalance();
          }, 3000);
        });
    },
    transfer() {
      // Confrim and send out a transfer.
      const evmAmount = utils.humanToEVM(this.toamount.toString(), 18);
      operations
        .transferToken(
          this.contract,
          this.myaddress,
          this.toaddress,
          evmAmount,
          this.toamount,
          "VTHO"
        )
        // eslint-disable-next-line no-unused-vars
        .then((result) => {
          alert("success!");
        })
        // eslint-disable-next-line no-unused-vars
        .catch((e) => {
          alert("failed!");
        });
    },
  },
};
</script>
