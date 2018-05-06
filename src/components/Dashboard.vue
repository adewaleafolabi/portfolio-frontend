<template>
  <dashboard-layout title="Dashboard">
    <div class="md-layout md-gutter md-alignment-top-center">
        <div class="md-layout-item">
          <md-button class="md-fab md-primary" @click="showDialog = true">
            <md-icon >add</md-icon>
          </md-button>
        </div>
        <div class="md-layout-item"></div>
        <div class="md-layout-item"></div>
        <div class="md-layout-item"><span style="float:right">{{portfolioFiat | formatNumber}}</span></div>
    </div>
      <pie-chart :donut="false" :data="pie" ></pie-chart>
      <div class="md-layout md-gutter md-alignment-top-center">
        <div class="md-layout-item">
            <md-button :to="`/asset/${p[0]}`"  v-for="(p, index) in pie" :key="index">{{p[0]}} <br/>{{p[1] | formatNumber}}</md-button>
        </div>
      </div>

      <div>
    <md-dialog-prompt
      :md-active.sync="showDialog"
      v-model="assetCode"
      md-title="New Asset"
      md-input-maxlength="30"
      md-input-placeholder="Asset Code"
      md-confirm-text="Done"
      @md-confirm="enableAsset" />
      
      <md-snackbar :md-active.sync="messageAvailable">{{message}}</md-snackbar>
    
   
  </div>
  </dashboard-layout>
</template>

<script>
import DashboardLayout from "@/components/DashboardLayout";
export default {
  data: () => ({
    assetCode:'',
    message: '',
    messageAvailable: false,
    showDialog:false,
    portfolioFiat:0,
    pieData: {},
    pie: []
  }),
  components: {
    DashboardLayout
  },
  methods: {
    enableAsset() {
      this.$http
        .post("http://localhost:3000/wallet", {
	"name": "",
	"code": this.assetCode,
	"balance": 0
},{
          headers: {
            authorization: "Bearer " + this.$localStorage.get("token")
          }
        })
        .then(response => {
          this.pie.push([this.assetCode,0])
          this.messageAvailable = true
          this.message = "Asset enabled"
        })
    },
    getFolio() {
      this.$http
        .get("http://localhost:3000/wallet", {
          headers: {
            authorization: "Bearer " + this.$localStorage.get("token")
          }
        })
        .then(response => {
          console.log(response)
           this.portfolioFiat =response.headers["total-fiat"]
          response.data.map(k => {
            this.pieData[k.code] = k.balance;
            var temp = [k.code, (k.balance*k.fiatPrice).toFixed(2)];
            this.pie.push(temp);
          });
        });
    }
  },
  mounted() {
    this.getFolio();
  }
};
</script>