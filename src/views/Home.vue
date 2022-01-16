<template>
  <div class="home">
    <map-maker msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import MapMaker from '@/components/MapMaker.vue';

export default {
  name: 'Home',
  components: {
    MapMaker,
  },
};
</script>
