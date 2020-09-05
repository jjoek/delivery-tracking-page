<template>
  <div id="app">
<!--      <button id="refresh_button" @click="">refresh</button>-->
    <delivery-view class="map"></delivery-view>
  </div>
</template>

<script>
import DeliveryView from './components/DeliveryView';

export default {
    name: 'app',

    components: {
        DeliveryView,
    },

    data() {
        return {
            msg: 'Welcome to Your Vue.js App',
        };
    },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

#refresh_button {
    background-color: #1f1f3e;
    padding: 10px 5px;
    position: absolute;
    z-index: 9999;
    color: white;
    margin-top: 60px;
    margin-left: 20px;
    border-radius: 10%;
}

.map {
height: 500px;
}
</style>
