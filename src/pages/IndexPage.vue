<template>
  <q-page class="py-4 flex flex-col items-center">
    <div class="w-full">
      <q-tab-panels
        v-model="panel"
        animated
        swipeable
        infinite
        style="height: 95vh"
        class="shadow-2 rounded-borders"
      >
        <q-tab-panel name="wifi">
          <WifiPage />
        </q-tab-panel>

        <q-tab-panel name="bluetooth">
          <BluetoothPage />
        </q-tab-panel>
      </q-tab-panels>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import WifiPage from "./WifiPage.vue";
import BluetoothPage from "./BluetoothPage.vue";
export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      panel: "bluetooth",
    };
  },
  components: {
    WifiPage,
    BluetoothPage,
  },

  mounted() {},
});
</script>
