<template>

  <div class="centered-container">
    <md-dialog-alert
      :md-active.sync="showModal"
      :md-content=modalText
      md-confirm-text="ok" />
    <md-content class="md-elevation-3">

      <!-- <div class="title">
        <img src="https://vuematerial.io/assets/logo-color.png">
        <div class="md-title">Vue Material</div>
        <div class="md-body-1">Build beautiful apps with Material Design and Vue.js</div>
      </div> -->
      

      <div class="form">
        <md-field>
          <label>Username</label>
          <md-input v-model="login.username" autofocus></md-input>
        </md-field>

        <md-field md-has-password>
          <label>Password</label>
          <md-input v-model="login.password" type="password"></md-input>
        </md-field>
      </div>

      <div class="actions md-layout md-alignment-center-space-between">
        <md-button class="md-raised md-primary" @click="auth">Log in</md-button>
      </div>

      <div class="loading-overlay" v-if="loading">
        <md-progress-spinner class="md-accent" md-mode="indeterminate"></md-progress-spinner>
        
      </div>

    </md-content>
    
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      showModal: false,
      modalText: '',
      login: {
        username: "",
        password: ""
      }
    };
  },
  methods: {
    auth() {
      // your code to login user
      // this is only for example of loading
      this.loading = true;
      this.showModal = false
      this.$http
        .post(`http://localhost:3000/login`, {
          username: this.login.username,
          password: this.login.password
        })
        .then(response => {
          this.loading = false;
          this.$localStorage.set("username", response.data.username);
          this.$localStorage.set("token", response.data.token);
          this.$localStorage.set("id", response.data.id);
          this.$router.push('Dashboard');
        }).catch(e => {
          console.log(e)
          if (e.response.data.message){
          this.modalText =e.response.data.message
            
          }else{
            this.modalText ='An error occurred'
              
          }
          this.loading = false
          this.showModal = true
        })
    }
  },
  mounted() {
    if (this.$localStorage.get("token") !== null) {
      this.$router.push({name: 'Dashboard'})
    }
  }
};
</script>

<style lang="scss">
.centered-container {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  height: 100vh;
  .title {
    text-align: center;
    margin-bottom: 30px;
    img {
      margin-bottom: 16px;
      max-width: 80px;
    }
  }
  .actions {
    .md-button {
      margin: 0;
    }
  }
  .form {
    margin-bottom: 60px;
  }
  .background {
    background-color: #fff;
    position: absolute;
    height: 100%;
    width: 100%;
    top: 0;
    bottom: 0;
    right: 0;
    left: 0;
    z-index: 0;
  }
  .md-content {
    z-index: 1;
    padding: 40px;
    width: 100%;
    max-width: 400px;
    position: relative;
  }
  .loading-overlay {
    z-index: 10;
    top: 0;
    left: 0;
    right: 0;
    position: absolute;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255, 255, 255, 0.9);
  }
    .md-progress-spinner {
    margin: 24px;
  }
}
</style>