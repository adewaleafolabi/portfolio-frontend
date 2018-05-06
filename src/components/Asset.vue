<template>
   <dashboard-layout :title="assetCode">
      
      <div class="md-layout md-gutter md-alignment-top-right">
         
         <div class="md-layout-item">
           <md-button class="md-fab md-primary" @click="showDialog = true">
         <md-icon>add</md-icon>
      </md-button>
         </div>
         <div class="md-layout-item center">
            <h1>{{asset.balance.toFixed(8)}} <small>{{assetCode}}</small></h1>
            <h4>{{asset.currency}} {{asset.balance * fiatPrice | formatNumber}}</h4>
         </div>
         <div class="md-layout-item"></div>

      </div>
      <div class="md-layout md-gutter ">
         <div class="md-layout-item">
            <md-table v-model="asset.transactions"  md-sort="date" md-sort-order="asc" md-card>
               <md-table-toolbar>
                  <div class="md-toolbar-section-start">
                     <h1 class="md-title">Transactions</h1>
                  </div>
                  <md-field md-clearable class="md-toolbar-section-end">
                     <md-input placeholder="Search..." v-model="search" @input="searchOnTable" />
                  </md-field>
               </md-table-toolbar>
               <md-table-empty-state
                  md-label="No transaction found"
                  :md-description="`No transaction found for this asset '${assetCode}'. `">
                  <md-button class="md-primary md-raised" >Create New Transaction</md-button>
               </md-table-empty-state>
               <md-table-row slot="md-table-row" slot-scope="{ item }">
                  <md-table-cell md-label="Date" md-sort-by="date">{{ item.date.substring(0,10) }}</md-table-cell>
                  <md-table-cell md-label="Price" md-sort-by="price" >{{ (item.amountPaid/item.quantity) | formatNumber }}</md-table-cell>
                  <md-table-cell md-label="Amount Paid" md-sort-by="amountPaid" >{{ item.amountPaid | formatNumber }}</md-table-cell>
                  <md-table-cell md-label="Currency" md-sort-by="currency" >{{ item.currency }}</md-table-cell>
                  <md-table-cell md-label="Quantity" md-sort-by="quantity" >{{ item.quantity }}</md-table-cell>
                  <md-table-cell md-label="Current Value" md-sort-by="currentValue" :class="[getNumberColorClass(item.growth)]" >{{ item.quantity * fiatPrice | formatNumber }}</md-table-cell>
                  <md-table-cell md-label="Profit" md-sort-by="profit" :class="[getNumberColorClass(item.growth)]">{{ (item.quantity * fiatPrice) - item.amountPaid | formatNumber }}</md-table-cell>
                  <md-table-cell md-label="Growth" md-sort-by="growth" :class="[getNumberColorClass(item.growth)]">{{ item.growth  }} %</md-table-cell>
                  <md-table-cell md-label="Action"  >
                    <md-button class="md-icon-button md-dense md-raised md-primary">
                      <md-icon>edit</md-icon>
                    </md-button>
                    <md-button class="md-icon-button md-dense md-raised md-accent" @click="transactionId = item.id;showWarning = true">
                      <md-icon>delete</md-icon>
                    </md-button>
                  </md-table-cell>
               </md-table-row>
            </md-table>
      <p>Total Invested: {{totalInvested.toFixed(2) | formatNumber}}</p>
      <p>Profit: {{((asset.balance * fiatPrice )-totalInvested).toFixed(2) | formatNumber}}</p>
      <p>Growth: {{(((asset.balance * fiatPrice )-totalInvested)/totalInvested).toFixed(2)*100}} %</p>
            
            <md-dialog-confirm
      :md-active.sync="showWarning"
      md-title="Delete Transaction?"
      md-content="Do you wan to <strong>delete</strong> the transaction ?"
      md-confirm-text="Ok"
      md-cancel-text="Cancel"
      @md-cancel="onCancel"
      @md-confirm="onDeleteConfirm" />
         </div>
      </div>

      <div class="">
        <md-dialog :md-active.sync="showDialog">
      <md-dialog-title>Transaction</md-dialog-title>
         <form novalidate class="form-dialog" @submit.prevent="validateUser">
            
                  <md-datepicker v-model="form.date"  :disabled="sending" :class="getValidationClass('date')">
                        <label>Transaction date</label>
                     </md-datepicker>
                     <md-field :class="getValidationClass('amountPaid')">
                        <label for="last-name">Total Paid</label>
                        <md-input name="last-name" id="last-name" type="number" v-model.number="form.amountPaid" :disabled="sending" />
                        <span class="md-error" v-if="!$v.form.amountPaid.required">The total paid is required</span>
                     </md-field>
                     <md-field :class="getValidationClass('quantity')">
                        <label for="quantity">Quantity Received</label>
                        <md-input type="number" id="quantity" name="quantity" autocomplete="quantity" v-model.number="form.quantity" :disabled="sending" />
                        <span class="md-error" v-if="!$v.form.quantity.required">The quantity received is required</span>
                     </md-field>
                     <md-field :class="getValidationClass('type')">
                        <label for="type">Type</label>
                        <md-select name="type" id="type" v-model="form.type" md-dense :disabled="sending">
                           <md-option value="BUY">Buy</md-option>
                           <md-option value="SELL">Sell</md-option>
                        </md-select>
                        <span class="md-error">The transaction type is required</span>
                     </md-field>
                     <md-field :class="getValidationClass('currency')">
                        <label for="currency">Curreny</label>
                        <md-select name="currency" id="currency" v-model="form.currency" md-dense :disabled="sending">
                           <md-option value="NGN">NGN</md-option>
                           <md-option value="CAD">CAD</md-option>
                           <md-option value="ETH">ETH</md-option>
                        </md-select>
                        <span class="md-error">The currency is required</span>
                     </md-field>
               <md-progress-bar md-mode="indeterminate" v-if="sending" />
               
           
            <md-snackbar :md-active.sync="userSaved">The transaction was saved successfully!</md-snackbar>
         </form>
          <md-dialog-actions>
        <md-button class="md-primary" @click="showDialog = false">Close</md-button>
        <md-button type="submit" class="md-primary" @click="validateUser" :disabled="sending">Create</md-button>
        
      </md-dialog-actions>
    </md-dialog>
      </div>
      <spinner-dialog :show="loading"></spinner-dialog>
   </dashboard-layout>
</template>

<style scoped>
.form-dialog{
  padding: 0px 20px
}
.md-menu-content {
  z-index: 110 !important;
}
.center{
  text-align: center
}
.pump {
  color: green;
  font-weight:bolder
}

.bleed{
  color: firebrick;
  font-weight: bolder;
}

</style>

<script>
import DashboardLayout from "@/components/DashboardLayout";
import SpinnerDialog from "@/components/SpinnerDialog";
 
import { validationMixin } from "vuelidate";
import {
  required,
  email,
  minLength,
  maxLength
} from "vuelidate/lib/validators";
export default {
  mixins: [validationMixin],
  data: () => ({
    transactionId: 0,
    showDialog:false,
    loading:false,
    showWarning:false,
    totalInvested: 0,
    btcPrice:0,
    fiatPrice:0,
    assetCode: "",
    currency: "",
    search: "",
    asset: {
      transactions: [],
      balance: 0,
      id: 0,
    },
    form: {
      date: null,
      amountPaid: 0,
      type: 'BUY',
      currency: 'NGN',
      quantity: 0,
      code: this.assetCode
    },
    userSaved: false,
    sending: false,
    lastUser: null
  }),
  validations: {
    form: {
      date: {
        required
      },
      amountPaid: {
        required
      },
      quantity: {
        required
      },
      type: {
        required
      },
      currency: {
        required
      }
    }
  },
  components: {
    DashboardLayout,
    SpinnerDialog
  },
  methods: {
    getNumberColorClass(number){
      if(number > 0)
        return "pump"
      return "bleed"
    },
    onDeleteConfirm () {
      this.deleteTransaction()
    },
    onCancel () {
      this.showWarning = false
    },
    searchOnTable() {},
    getBTCETHPrice() {
      this.$http
        .get(`http://localhost:3000/wallet/exchange`,{
          headers: {
            authorization: "Bearer " + this.$localStorage.get("token")
          }
        })
        .then(response => {
          this.btcPrice = parseFloat(response.data.filter(k=>k.pair==="XBTNGN")[0].last_trade)
          this.ethPrice = this.btcPrice * parseFloat(response.data.filter(k=>k.pair==="ETHXBT")[0].last_trade)
          // this.asset.balance = response.data.balance;
          // this.asset.transactions = response.data.transactions.map(k => {
          //   return {
          //     date: k.date,
          //     amountPaid: k.amountPaid,
          //     quantity: k.quantity
          //   };
          // });
        });

    },
    getTransactions() {
      this.loading = true
      this.$http
        .get(`http://localhost:3000/wallet/${this.assetCode}`, {
          headers: {
            authorization: "Bearer " + this.$localStorage.get("token")
          }
        })
        .then(response => {
          this.loading = false
          this.btcPrice = response.data.btcPrice
          this.fiatPrice = response.data.fiatPrice
          this.asset.balance = response.data.balance;
          this.asset.currency = response.data.currency;
          this.asset.id = response.data.id;
          
          this.asset.transactions = response.data.transactions.map(k => {
            this.totalInvested+=k.amountPaid
            return {
              currency: k.currency,
              date: k.date,
              amountPaid: k.amountPaid,
              quantity: k.quantity,
              id: k.id,
              growth: ((((k.quantity * this.fiatPrice) - k.amountPaid) / k.amountPaid)*100).toFixed(2)
            };
          });
        });
    },
    saveTransaction() {
      this.$http
        .post(
          `http://localhost:3000/wallet/${this.assetCode}/transactions`, this.form,
          {
            headers: {
              authorization: "Bearer " + this.$localStorage.get("token")
            }
          }
        )
        .then(response => {
          console.log(response.data);
          this.asset.balance = response.data.balance;
          this.asset.transactions.push({
            date: response.data.date,
            amountPaid: response.data.amountPaid,
            quantity: response.data.quantity,
            currency: response.data.currency
          });
          this.userSaved = true;
          this.sending = false;
          this.clearForm();
        });
    },
     deleteTransaction() {
       this.loading = true
      this.$http
        .delete(
          `http://localhost:3000/wallet/${this.assetCode}/transactions/${this.transactionId}`, 
          {
            headers: {
              authorization: "Bearer " + this.$localStorage.get("token")
            }
          }
        )
        .then(response => {
          this.loading = false
          this.getTransactions()
        });
    },
    getValidationClass(fieldName) {
      const field = this.$v.form[fieldName];

      if (field) {
        return {
          "md-invalid": field.$invalid && field.$dirty
        };
      }
    },
    closeDialog() {
      this.showDialog = false
    },
    clearForm() {
      this.$v.$reset();
      this.form.date = null;
      this.form.amountPaid = 0;
      this.form.quantity = 0;
      this.form.type = 'BUY';
      this.form.currency = 'NGN';
    },
    saveUser() {
      this.sending = true;

      // Instead of this timeout, here you can call your API
      // window.setTimeout(() => {
      //   this.lastUser = `${this.form.date} ${this.form.amountPaid}`
      //   this.userSaved = true
      //   this.sending = false
      //   this.clearForm()
      // }, 1500)
    },
    validateUser() {
      this.$v.$touch();

      if (!this.$v.$invalid) {
        this.saveTransaction();
      } else {
        console.log("Invalid body");
      }
    }
  },
  mounted() {
    this.assetCode = this.$route.params.code;
    this.getTransactions();
  }
};
</script>


