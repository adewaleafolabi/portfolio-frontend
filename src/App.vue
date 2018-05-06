<template>
  <div id="app">
    <router-view/>
  </div>
</template>
<style>
#nprogress .bar {
  background:orange;
}
.md-menu-content.md-select-menu{
  z-index: 100
}
</style>

<script>
export default {
  name: "App"
};
</script>


