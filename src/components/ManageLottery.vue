<template>
    <div class="row">
        <div class="col-md-12">
            <div class="card">
                <h5 class="card-header">Manage Lottery</h5>
                <div class="card-body">
                    <div v-if="isLotteryLive">
                        <button style="margin:auto" class="btn btn-primary" @click="declareWinner" :disabled="isDisabled">Declare winner</button>
                    </div>

                    <div v-else>
                        <div class="row">
                          <button style="margin:auto" class="btn btn-danger" @click="deleteLottery">Delete Lottery</button>
                        </div>
                        <div class="row voffset4">
                          <form style="margin: auto" @submit.prevent="activateLottery">
                              <div class="form-group">
                                  <label for="maxEntriesPerPlayer"> Max Entries for a player</label>
                                  <input class="form-control"
                                      type="number" id="maxEntriesPerPlayer"
                                      v-model="maxEntriesPerPlayer">
                              </div>
                              <div class="form-group">
                                  <label for="ethRequiredToParticipate"> Amount of Ether required to participate </label>
                                  <input
                                    class="form-control"
                                    id="ethRequiredToParticipate"
                                    @keypress="isNumber($event)"
                                    v-model="ethRequiredToParticipate"
                                  >
                              </div>
                              <button class="btn btn-success" type="submit">Activate</button>
                          </form>
                        </div>
                    </div>

                    <div v-if="showError" class="alert alert-danger voffset2" role="alert">
                        There was an error while making transaction. Try again later.
                    </div>
                    <div
                      v-if="showProgress"
                      class="progress-bar progress-bar-striped progress-bar-animated"
                      aria-valuemin="0"
                      aria-valuemax="100"
                      :style="{'width': '100%'}"
                    >
                        Choosing winner...
                    </div>
                    <div
                      v-if="deleteShowProgress"
                      class="progress-bar progress-bar-striped progress-bar-animated"
                      aria-valuemin="0"
                      aria-valuemax="100"
                      :style="{'width': '100%'}"
                    >
                        Deleting lottery...
                    </div>
                    <div
                      v-if="activeShowProgress"
                      class="progress-bar progress-bar-striped progress-bar-animated"
                      aria-valuemin="0"
                      aria-valuemax="100"
                      :style="{'width': '100%'}"
                    >
                        Activating lottery...
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Lottery from "../web3/Lottery";
import LotteryGenerator from "../web3/LotteryGenerator";

export default {
  props: [
    "lotteryAddress",
    "isLotteryLive",
    "accounts",
    "changeLotteryStatus",
    "updateWinner",
    "createdAt"
  ],
  data() {
    return {
      maxEntriesPerPlayer: 0,
      ethRequiredToParticipate: 0,
      showError: false,
      showProgress: false,
      deleteShowProgress: false,
      activeShowProgress: false,
      progress: 0,
      currentTime: null,
      remainTime: null,
      timeCounter: null
    };
  },
  computed: {
  	isDisabled: function() {
    	return this.remainTime != "End";
    }
  },
  methods: {
    isNumber: function(evt) {
      evt = (evt) ? evt : window.event;
      var charCode = (evt.which) ? evt.which : evt.keyCode;
      if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
        evt.preventDefault();;
      } else {
        return true;
      }
    },
    getCurrentTime () {
      var currentDate = new Date();
      var endDate = new Date(this.createdAt);
      endDate.setMinutes(endDate.getMinutes() + 5);
      var diff = endDate.getTime() - currentDate.getTime();
      if (diff <= 0) {
        this.remainTime = "End";
        clearInterval(this.timeCounter);
        if (diff == 0) this.declareWinner();
      } else {
        diff = diff / 1000;
        var diffH = Math.floor(diff / 3600);
        var diffM = Math.floor((diff - 3600 * diffH) / 60);
        var diffS = Math.floor(diff - diffH * 3600 - diffM * 60);
        this.remainTime = `${diffH}Hours ${diffM}Minutes ${diffS}Seconds`;
        this.currentTime = currentDate;
      }
    },

    activateLottery() {
      Lottery.methods
        .activateLottery(
          this.maxEntriesPerPlayer,
          this.ethRequiredToParticipate * 1000000000000000000
        )
        .send({
          from: this.accounts[0],
          gas: 2000000
        })
        .once("transactionHash", hash => {
          this.activeShowProgress = true;
        })
        .on("error", error => {
          console.log(error);
          this.showError = true;
        })
        .then(reciept => {
          this.activeShowProgress = false;
          this.changeLotteryStatus(true);
        });
    },

    declareWinner() {
      Lottery.methods
        .declareWinner()
        .send({
          from: this.accounts[0],
          gas: 200000
        })
        .once("transactionHash", hash => {
          this.showProgress = true;
        })
        .on("error", error => {
          console.log(error);
          this.showError = true;
        })
        .then(reciept => {
          this.showProgress = false;
          this.changeLotteryStatus(false);
          this.updateWinner(reciept.events.WinnerDeclared.returnValues);
        });
    },

    deleteLottery() {
      LotteryGenerator.methods
        .deleteLottery(this.lotteryAddress)
        .send({
          from: this.accounts[0],
          gas: 200000
        })
        .once("transactionHash", hash => {
          this.deleteShowProgress = true;
        })
        .on("error", error => {
          console.log(error);
          this.showError = true;
        })
        .then(reciept => {
          this.deleteShowProgress = false;
          this.$router.push('/');
        });
    }
  },
  created() {
    Lottery.options.address = this.lotteryAddress;
    this.timeCounter = setInterval(this.getCurrentTime, 1000);
  },
  destroyed() {
    clearInterval(this.timeCounter);
  }
};
</script>

<style>

</style>
