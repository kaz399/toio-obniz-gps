<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div v-if="isSupportedGeolocation">
      <h2>Input Your OBNIZ ID</h2>
      <input v-model="obniz_id" placeholder="YOUR OBNIZE ID" />
      <p>ID: {{ obniz_id }}</p>
      <div v-if="checkObnizId">
        <p>ID check OK</p>
        <div>
          <button v-on:click="connectToObniz">せ つ ぞ く</button>
          <button v-on:click="disconnectFromObniz">せ つ だ ん</button>
          <br />
          <br />
          <br />
          <button v-on:click="getLocation">現在地取得</button>
          <p>緯度: {{ latitude }}</p>
          <p>経度: {{ longitude }}</p>
        </div>
      </div>
      <div v-else>
        <p>This id is invalid!</p>
      </div>
    </div>
    <div v-else>
      <h2>このブラウザはGeolocationがサポートされていません</h2>
    </div>
  </div>
</template>

<script>
import Obniz from 'obniz';

const target_cube = {
  localName: "toio Core Cube-G9F",
}

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      obniz_id: "",
      obniz: null,
      cube: null,
      location: null,
      latitude: null,
      longitude: null,
    }
  },
  beforeUnmount: function () {
    if (this.obniz) {
      console.log('close connection');
      this.obniz.close();
    }
  },
  computed: {
    checkObnizId: function () {
      const id_regex = new RegExp(/^[0-9]{4}-[0-9]{4}$/);
      return id_regex.test(this.obniz_id);
    },
    isSupportedGeolocation: function () {
      return navigator.geolocation
    }
  },
  methods: {
    connectToObniz: function () {
      this.obniz = new Obniz(this.obniz_id);
      this.obniz.on('connect', async () => {
        console.log('connect to obnize');
        await this.connectToCube();
      });
    },
    disconnectFromObniz: async function () {
      console.log('close');
      await this.disconnectFromCube();
      this.obniz.close();
    },
    getLocation: function () {
      navigator.geolocation.getCurrentPosition((position) => {
        console.log(position);
        this.location = position;
        this.latitude = position.coords.latitude;
        this.longitude = position.coords.longitude;
        this.obniz.display.print('lat:' + this.latitude);
        this.obniz.display.print('lng:' + this.longitude);
      }, (error) => {
        console.log(error);
      });
      return this.location;
    },
    connectToCube: async function () {
      if (this.obniz) {
        await this.obniz.ble.initWait();
        const Toio_CoreCube = Obniz.getPartsClass("toio_CoreCube");
        this.obniz.ble.scan.onfind = async (peripheral) => {
          if (Toio_CoreCube.isDevice(peripheral)) {
            console.log("find toio core cube");
            this.cube = new Toio_CoreCube(peripheral);
            console.log("connecting...");
            await this.obniz.ble.scan.endWait();
            await this.cube.connectWait();
            console.log("connected");
          }
        };
        await this.obniz.ble.scan.startOneWait(target_cube);
      }
    },
    disconnectFromCube: async function () {
      if (this.cube) {
        await this.obniz.ble.scan.endWait();
        await this.cube.disconnectWait();
        this.cube = null;
        console.log("disconnected");
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
