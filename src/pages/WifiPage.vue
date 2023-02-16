<template>
  <div class="w-full">
    <div class="mt-4 text-xl font-bold uppercase">Redes WIFI Disponibles</div>
    <div class="flex flex-col">
      <q-list bordered separator>
        <template v-for="wifi in wifiList" :key="wifi.BSSID">
          <q-item clickable v-ripple class="flex">
            <q-item-section>{{ wifi.SSID }}</q-item-section>
            <q-circular-progress
              show-value
              color="green"
              track-color="grey-3"
              :value="100 + wifi.level"
              size="30px"
            />
          </q-item>
        </template>
      </q-list>
    </div>
    <div class="w-full flex justify-between px-3 my-4">
      <q-btn color="primary" icon="refresh" @click="getWifiList"
        >Escanear</q-btn
      >
    </div>
    <div class="flex flex-col px-2 mt-6">
      <div class="mt-4 text-xl font-bold uppercase">
        Detalles de la red actual
      </div>
      <q-list bordered separator>
        <q-item clickable v-ripple class="flex">
          <q-item-section class="font-bold">SSID:</q-item-section>
          <q-item-section>{{ currentWifi.SSID }}</q-item-section>
        </q-item>
        <q-item clickable v-ripple class="flex">
          <q-item-section class="font-bold">IP:</q-item-section>
          <q-item-section>{{ currentWifi.ip }}</q-item-section>
        </q-item>
        <q-item clickable v-ripple class="flex">
          <q-item-section class="font-bold">ROUTER:</q-item-section>
          <q-item-section>{{ currentWifi.router }}</q-item-section>
        </q-item>
        <q-item clickable v-ripple class="flex">
          <q-item-section class="font-bold">SUBNET:</q-item-section>
          <q-item-section>{{ currentWifi.subnet }}</q-item-section>
        </q-item>
        <q-item clickable v-ripple class="flex">
          <q-item-section class="font-bold">MASK:</q-item-section>
          <q-item-section>{{ currentWifi.BSSID }}</q-item-section>
        </q-item>
      </q-list>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";
import { Notify } from "quasar";
import WifiWizard2 from "app/src-cordova/plugins/wifiwizard2/www/WifiWizard2";
export default defineComponent({
  name: "WifiPage",
  data() {
    return {
      wifiList: [],
      currentWifi: {},
    };
  },

  methods: {
    async getWifiList() {
      try {
        this.wifiList = [];

        await WifiWizard2.requestPermission().then(async (res) => {
          if (res === "PERMISSION_GRANTED") {
            await WifiWizard2.scan().then((res) => {
              this.wifiList = res;
            });
          } else {
            console.log("Permission Denied");
          }
        });
      } catch (error) {
        Notify.create({
          message: error,
          color: "negative",
        });
        this.loading = false;
      }
    },
    async getCurrentWifi() {
      try {
        this.currentWifi = {};
        await WifiWizard2.getConnectedSSID().then((res) => {
          this.currentWifi.SSID = res;
        });
        await WifiWizard2.getConnectedBSSID().then((res) => {
          this.currentWifi.BSSID = res;
        });
        await WifiWizard2.getWifiIPInfo().then((res) => {
          this.currentWifi.ip = res.ip;
          this.currentWifi.subnet = res.subnet;
        });
        await WifiWizard2.getWifiRouterIP().then((res) => {
          this.currentWifi.router = res;
        });
      } catch (error) {
        Notify.create({
          message: JSON.stringify(error),
          color: "negative",
        });
        this.loading = false;
      }
    },
  },
  async mounted() {
    document.addEventListener(
      "deviceready",
      async () => {
        this.getWifiList();
        this.getCurrentWifi();
      },
      false
    );
  },
});
</script>
