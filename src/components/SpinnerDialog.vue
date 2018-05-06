<template>

<div class="loading-overlay" v-if="show">
        <md-progress-spinner class="md-accent" md-mode="indeterminate"></md-progress-spinner>
        
      </div>
  
</template>

<script>
export default {
  props: ['show']
}
</script>
<style scoped>
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
</style>
