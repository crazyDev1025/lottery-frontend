<template>
  <div class="row">
    <div class="col-md-12">
      <div class="card">
        <h5 class="card-header">Participate - Max ({{maxEntriesForPlayer}})</h5>
        <div class="card-body">
          <p>Current winning price: {{currentWinningPrice}} eth</p>
          <form @submit.prevent="participate">
            <div class="form-group">
              <input
                type="text"
                class="form-control"
                id="playerName"
                placeholder="Input your name"
                v-model="playerName"
                required>
            </div>
            <div class="form-group">
              <label for="ethToParticipate">Ether required to participate</label>
              <input
                type="text"
                class="form-control"
                id="ethToParticipate"
                v-model="ethToParticipate"
                disabled>
            </div>
            <button type="submit" class="btn btn-primary" :disabled="isEnd">Participate</button>
            <div v-if="error" class="alert alert-danger voffset2" role="alert">
              {{error}}
            </div>
            <div
              v-if="showProgress"
              class="progress-bar progress-bar-striped progress-bar-animated mt-3"
              aria-valuemin="0"
              aria-valuemax="100"
              :style="{'width': '100%'}"
            >
              Processing...
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { eventBus } from "../main";

import web3 from "../web3/web3";
import Lottery from "../web3/Lottery";

export default {
  props: ["lotteryAddress", "accounts", "isDisabled"],
  data() {
    return {
      currentWinningPrice: 0,
      ethToParticipate: 0,
      maxEntriesForPlayer: 0,
      player: {},
      playerName: "",
      error: null,
      showProgress: false,
      isEnd: false
    };
  },
  methods: {
    participate() {
      var weiToParticipate = parseFloat(this.ethToParticipate) * 1000000000000000000;
      if (this.maxEntriesForPlayer - this.player[1] <= 0) {
        this.error = "You have reached participation limit";
        return;
      }
      Lottery.methods
        .participate(this.playerName)
        .send({
          from: this.accounts[0],
          gas: "1000000",
          value: weiToParticipate
        })
        .once("transactionHash", hash => {
          this.showProgress = true;
        })
        .on("error", error => {
          console.log(error);
          this.error =
            "There was an error while making transaction, please try again later.";
        })
        .then(reciept => {
          eventBus.$emit(
            "playerParticipated",
            reciept.events.PlayerParticipated.returnValues
          );
          this.currentWinningPrice = parseFloat(this.ethToParticipate) + parseFloat(this.currentWinningPrice);
          this.showProgress = false;
        });
    }
  },
  created() {
    Lottery.options.address = this.lotteryAddress;
    Lottery.methods
      .ethToParticipate()
      .call()
      .then(result => {
        this.ethToParticipate = result / 1000000000000000000;
      });
    Lottery.methods
      .maxEntriesForPlayer()
      .call()
      .then(result => {
        this.maxEntriesForPlayer = result;
      });
    Lottery.methods
      .getWinningPrice()
      .call()
      .then(result => {
        this.currentWinningPrice = web3.utils.fromWei(result, "ether");
      });
    Lottery.methods
      .getPlayer(this.accounts[0])
      .call()
      .then(player => {
        this.player = player;
      });
  },
  mounted() {
    this.isEnd = this.isDisabled;
  }
};
</script>

<style>

</style>
