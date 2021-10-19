<template>
  <div class="card">
    <div class="card-body">
      <form @submit.prevent="createLottery">
        <div class="form-group">
          <label for="lotteryName">Create a Lottery</label>
          <input
            type="text"
            class="form-control"
            id="lotteryName"
            placeholder="Enter Lottery name"
            v-model="lotteryName"
            required> 
        </div>
        <button type="submit" class="btn btn-success">Create</button>
      </form>
      <div v-if="showError" class="alert alert-danger voffset2" role="alert">
        There was an error while creating lottery, try again later.
      </div>
      <div v-if="isProgress" class="progress voffset2">
        <div
          class="progress-bar progress-bar-striped progress-bar-animated"
          aria-valuemin="0"
          aria-valuemax="100"
          :style="{'width': '100%'}"
        >
          Creating...
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import web3 from "../web3/web3";
import LotteryGenerator from "../web3/LotteryGenerator";

export default {
  props: ["lotteryCreated", "accounts"],
  data: function() {
    return {
      lotteryName: "",
      showError: false,
      isProgress: false,
    };
  },
  methods: {
    async createLottery() {
      var currentDate = new Date();
      const year = currentDate.getFullYear();
      const month = (currentDate.getMonth()+1).toString().padStart(2, '0');
      const date = currentDate.getDate().toString().padStart(2, '0');
      const time = currentDate.getHours().toString().padStart(2, '0') + ':' + currentDate.getMinutes().toString().padStart(2, '0') + ':' + currentDate.getSeconds().toString().padStart(2, '0');
      const strDate = `${year}-${month}-${date} ${time}`;
      LotteryGenerator.methods
        .createLottery(this.lotteryName, strDate)
        .send({
          gas: 2000000,
          from: this.accounts[0]
        })
        .once('transactionHash', (hash)=> {
          this.isProgress = true;
        })
        .on('error', (error)=>{
          console.log(error);
          this.showError = true
        })
        .then((reciept) => {
          this.isProgress = false;
          this.lotteryCreated(reciept.events.LotteryCreated.returnValues.lotteryAddress)
        });
    }
  },
  mounted() {
  }
};
</script>

<style>

</style>
