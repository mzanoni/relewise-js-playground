<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <header>
    <div class="wrapper">
      <h1>Relewise JS Playground</h1>
    </div>
  </header>

  <RouterView />
</template>

<style scoped>
.wrapper {
  padding: 1rem;
  padding-bottom: 0;
  padding-top: 0rem;
}
header {
  line-height: 1.3;
}

h1 {
  margin: 0;
  font-weight: bold;
  font-size: 35px;

  display: inline-block;
  /*
* Created with https://www.css-gradient.com
* Gradient link: https://www.css-gradient.com/?c1=57c8ea&c2=02977a&gt=l&gd=dcr
*/
  background: #57C8EA;
  background: linear-gradient(270deg, #57C8EA, #02977A);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>
