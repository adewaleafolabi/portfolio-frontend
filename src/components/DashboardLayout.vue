<template>
  <div class="page-container">
    <md-app md-mode="reveal">
      <md-app-toolbar class="md-primary">
        <md-button class="md-icon-button" @click="menuVisible = !menuVisible">
          <md-icon>menu</md-icon>
          
        </md-button>
        
        <span class="md-title">{{title}}</span>
         <!--<div class="md-layout md-gutter md-alignment-top-right">
            <div class="md-layout-item">
        <md-field>
          <label for="movie">Currency</label>
          <md-select v-model="currency" name="movie" id="movie">
            <md-option value="NGN">NGN</md-option>
            <md-option value="BTC">BTC</md-option>
            <md-option value="USD">USD</md-option>
            <md-option value="CAD">CAD</md-option>
            <md-option value="ETH">ETH</md-option>
          </md-select>
        </md-field>
      </div>
         </div>-->
      </md-app-toolbar>

      <md-app-drawer :md-active.sync="menuVisible">
        <md-toolbar class="md-transparent" md-elevation="0">Navigation</md-toolbar>

        <md-list>
          <md-list-item @click="redirect('Dashboard')">
            <md-icon>home</md-icon>
            <span class="md-list-item-text">Home</span>
          </md-list-item>

          <md-list-item>
            <md-icon>send</md-icon>
            <span class="md-list-item-text">Sent Mail</span>
          </md-list-item>

          <md-list-item>
            <md-icon>delete</md-icon>
            <span class="md-list-item-text">Trash</span>
          </md-list-item>

          <md-list-item @click="logout()">
            <md-icon>exit_to_app</md-icon>
            <span class="md-list-item-text">Logout</span>
          </md-list-item>
        </md-list>
      </md-app-drawer>

      <md-app-content>
          
          <slot></slot>
          <md-button class="md-fab md-mini md-plain md-fab-top-right "  @click="redirect('Dashboard')">
            <md-icon>home</md-icon>
          </md-button>
          
      </md-app-content>
    </md-app>
  </div>
</template>

<style lang="scss" scoped>
.md-app {
  min-height: 700px;
  border: 1px solid rgba(#000, 0.12);
}

//    // Demo purposes only
//   .md-drawer {
//     width: 230px;
//     max-width: calc(100vw - 125px);
//   }
</style>

<script>
export default {
  data: () => ({
    menuVisible: false
  }),
  props: ["title"],
  methods: {
    redirect (name) {
      this.$router.push({name: name})
    },
    logout () {
      this.$localStorage.remove("username")
          this.$localStorage.remove("token");
          this.$localStorage.remove("id");
          this.$router.push({name: 'Login'});
    }
  }
};
</script>
