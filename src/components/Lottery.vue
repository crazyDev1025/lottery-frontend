<template>
<div>
  <div class="row">
    <div class="col-md-4 offset-md-4" v-if="winner.name">
      <div class="card">
        <h6 class="card-header">Last Winner is: {{winner.name}}</h6>
      </div>
    </div>
    <div class="col-md-4 offset-md-4">
      <div class="alert alert-danger" v-if="isDisabled">Close after: {{this.remainTime}}</div>
      <div class="alert alert-danger" v-if="!isDisabled">{{this.remainTime}}</div>
    </div>
  </div>
  <div class="row voffset4" v-if="isLotteryLive">
    <div class="col-md-6">
      <participants
        :lotteryAddress="lotteryAddress"
        :accounts="accounts"
      ></participants>
    </div>
    <div class="col-md-6">
      <participate
        :lotteryAddress="lotteryAddress"
        :accounts="accounts"
        :isDisabled="isDisabled"
      ></participate>
    </div>
  </div>
  <div class="row voffset3" v-if="isManager()">
    <div class="col-md-6 offset-md-3">
      <manager-lottery
        :lotteryAddress="lotteryAddress"
        :accounts="accounts"
        :isLotteryLive="isLotteryLive"
        :changeLotteryStatus="changeLotteryStatus"
        :updateWinner="updateWinner"
        :createdAt="createdAt"
        :isDisabled="isDisabled"
      ></manager-lottery>
    </div>
  </div>
</div>
</template>

<script>
import Vue from "vue";
import Participate from "./Participate";
import Participants from "./Participants";
import ManagerLottery from "./ManageLottery";

import web3 from "../web3/web3";

import Lottery from "../web3/Lottery";

export default {
  components: {
    Participate,
    Participants,
    ManagerLottery
  },
  data() {
    return {
      lotteryAddress: this.$route.params.lotteryAddress,
      createdAt: this.$route.params.createdAt,
      winner: {},
      accounts: [],
      isLotteryLive: false,
      lotteryManager: "",
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
    getCurrentTime () {
      var currentDate = new Date();
      var endDate = new Date(this.createdAt);
      endDate.setMinutes(endDate.getMinutes() + 5);
      var diff = endDate.getTime() - currentDate.getTime();
      if (diff <= 0) {
        this.remainTime = "End";
        clearInterval(this.timeCounter);
      } else {
        diff = diff / 1000;
        var diffH = Math.floor(diff / 3600);
        var diffM = Math.floor((diff - 3600 * diffH) / 60);
        var diffS = Math.floor(diff - diffH * 3600 - diffM * 60);
        this.remainTime = `${diffH}Hours ${diffM}Minutes ${diffS}Seconds`;
        this.currentTime = currentDate;
      }
    },
    isManager() {
      return this.lotteryManager === this.accounts[0];
    },
    changeLotteryStatus(status) {
      this.isLotteryLive = status;
    },
    updateWinner(winner) {
      this.winner = winner;
    }
  },
  created() {
    this.timeCounter = setInterval(this.getCurrentTime, 1000);
    Lottery.options.address = this.lotteryAddress;
    Lottery.methods
      .winner()
      .call()
      .then(result => {
        this.winner = result;
      });
    web3.eth.getAccounts().then(metaMaskAccounts => {
      this.accounts = metaMaskAccounts;
    });

    Lottery.methods
      .isLotteryLive()
      .call()
      .then(status => {
        this.isLotteryLive = status;
      });

    Lottery.methods
      .manager()
      .call()
      .then(managerName => {
        this.lotteryManager = managerName;
      });
  },
  destroyed() {
    clearInterval(this.timeCounter);
  },
  updated() {
  }
};
</script>

<style>

</style>
