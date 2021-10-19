<template>
  <div class="col-md-4" style="padding-top: 15px; padding-bottom:15px;">
    <div class="card">
      <h6 class="card-header">{{lotteryName}}</h6>
      <div class="card-body">
        <h6 class="card-title">{{'Created : ' + createdAt}}</h6>
        <h6 class="card-title">{{'End : ' + endAt}}</h6>
        <hr/>
        <div class="row">
          <div class="col-md-6">
              <router-link
                :to="{
                  name: 'Lottery',
                  params: {lotteryAddress: lotteryAddress, createdAt: createdAt}
                }"
                class="btn btn-primary"
              >
                {{isMyLottery() ? 'Manage' : 'Details'}}
              </router-link>
          </div>
          <div class="col-md-6 lottery-status">
            <span
              class="lottery-status-icon"
              :class="isLotteryLive? 'lottery-status-icon-green': 'lottery-status-icon-red'"
            ></span>
              {{isLotteryLive? 'Live': 'Closed'}}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Lottery from "../web3/Lottery";
import web3 from "../web3/web3";

var accounts = [];

export default {
  props: ["lotteryAddress", "accounts"],
  data() {
    return {
      lotteryName: "",
      lotteryManager: "",
      createdAt: "Created Date",
      endAt: "End Date",
      isLotteryLive: false
    };
  },
  methods: {
    isMyLottery() {
      return this.accounts[0] === this.lotteryManager;
    }
  },
  async created() {
    Lottery.options.address = this.lotteryAddress;

    Lottery.methods
      .lotteryName()
      .call()
      .then(name => {
        this.lotteryName = name;
      });

    Lottery.methods
      .manager()
      .call()
      .then(managerAddress => {
        this.lotteryManager = managerAddress;
      });

    Lottery.methods
      .createdAt()
      .call()
      .then(createdAt => {
        this.createdAt = createdAt;
        var endDate = new Date(this.createdAt);
        endDate.setMinutes(endDate.getMinutes() + 5);
        const year = endDate.getFullYear();
        const month = (endDate.getMonth()+1).toString().padStart(2, '0');
        const date = endDate.getDate().toString().padStart(2, '0');
        const hour = endDate.getHours().toString().padStart(2, '0');
        const minute = endDate.getMinutes().toString().padStart(2, '0');
        const second = endDate.getSeconds().toString().padStart(2, '0');
        const strDate = `${year}-${month}-${date} ${hour}:${minute}:${second}`;
        this.endAt = strDate;
      });

    Lottery.methods
      .isLotteryLive()
      .call()
      .then(status => {
        this.isLotteryLive = status;
      });
  }
};
</script>

<style>
.lottery-status {
  margin: auto;
}
.lottery-status-icon {
  width: 20px;
  height: 20px;
  border-radius: 10px;
  display: inline-block;
  vertical-align: text-bottom;
}

.lottery-status-icon-red {
  background-color: #e5003a;
}

.lottery-status-icon-green {
  background-color: green;
}

.my-lottery-color {
  background-color: lightgreen;
}
</style>
