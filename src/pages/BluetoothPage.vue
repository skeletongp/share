<template>
  <div class="w-full">
    <div class="mt-4 text-xl font-bold uppercase">
      Comprimir y Compartir Imágenes
    </div>
    <div class="flex flex-col">
      <div class="w-full flex flex-col justify-center px-3 my-4">
        <template v-if="image.url">
          <div class="flex flex-col">
            <q-img :src="image.url" :ratio="4 / 3" class="my-2">
              <div
                class="absolute-bottom text-subtitle1 text-center my-2"
                dense
                @click="chooseFile"
              >
                <q-btn> {{ image.name }}/ {{ image.size }}</q-btn>
              </div>
            </q-img>

            <div class="flex flex-col justify-between">
              <q-btn push color="primary" label="Comprimir Foto">
                <q-popup-proxy>
                  <q-banner class="bg-white">
                    <q-input
                      type="number"
                      v-model="percent"
                      placeholder="Ingrese porcentaje max(50)"
                      outlined
                      dense
                      :readonly="!image.size"
                    >
                      <template v-slot:append>
                        <q-btn
                          dense
                          flat
                          size="sm"
                          @click="zip"
                          :disabled="percent <= 0"
                        >
                          <q-icon name="compress" />
                        </q-btn>
                      </template>
                    </q-input>
                  </q-banner>
                </q-popup-proxy>
              </q-btn>

              <div class="flex justify-around my-3">
                <q-btn
                  :disabled="!image.size"
                  fab
                  dense
                  @click="sendFile(image.file, 'bluetooth')"
                  color="primary"
                >
                  <q-icon name="bluetooth" />
                </q-btn>
                <q-btn
                  :disabled="!image.size"
                  fab
                  dense
                  @click="sendFile(image.file, 'whatsapp')"
                  color="green"
                >
                  <q-icon class="ri-whatsapp-line" />
                </q-btn>
                <q-btn
                  :disabled="!image.size"
                  fab
                  dense
                  @click="sendFile(image.file, 'facebook')"
                  color="blue"
                >
                  <q-icon class="ri-facebook-fill" />
                </q-btn>
                <q-btn
                  :disabled="!image.size"
                  fab
                  dense
                  @click="sendFile(image.file, 'wifi')"
                  color="blue"
                >
                  <q-icon class="ri-wifi-fill" />
                </q-btn>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>
    <q-inner-loading :showing="loading" />
  </div>
</template>

<script>
import { defineComponent } from "vue";
import { Loading, Notify } from "quasar";
export default defineComponent({
  name: "BluetoothPage",

  data() {
    return {
      image: {
        file: "https://st3.depositphotos.com/23594922/31822/v/600/depositphotos_318221368-stock-illustration-missing-picture-page-for-website.jpg",
        url: "https://st3.depositphotos.com/23594922/31822/v/600/depositphotos_318221368-stock-illustration-missing-picture-page-for-website.jpg",
        name: "Elige una imagen",
      },
      percent: null,
      loading: false,
    };
  },
  methods: {
    async zip() {
      this.loading = true;
      if (this.percent > 50) {
        this.percent = 50;
      }
      await this.reduceImage(this.image.url)
        .then(async (res) => {
          var base64data = await new Promise((resolve, reject) => {
            var reader = new FileReader();
            reader.readAsDataURL(res);
            reader.onload = function () {
              resolve(reader.result);
            };
            reader.onerror = reject;
          });

          const size = res.size / 1e6;
          this.image.url = URL.createObjectURL(res);
          this.image.size = size.toFixed(2) + " MB";
          this.image.file = base64data;
          this.percent = null;
          console.log(this.image);
          this.loading = false;
          Notify.create({
            message: `Imagen comprimida al ${this.percent}%`,
            color: "positive",
          });
        })
        .catch((err) => {
          this.loading = false;
          console.log(err);
          Notify.create({
            message: err,
            color: "negative",
          });
        });
    },
    async sendFile(file, via = "bluetooth") {
      try {
        const ctx = this;
        this.loading = true;
        function onSuccess(result) {
          ctx.loading = false;
          Notify.create({
            message: "Imagen compartida via Bluetooth",
            color: "positive",
          });
        }

        function onError(msg) {
          ctx.loading = false;
          Notify.create({
            message: msg,
            color: "negative",
          });
        }

        var options = {
          message: "Imagen",
          files: [file],
          appPackageName: via,
        };

        window.plugins.socialsharing.shareWithOptions(
          options,
          onSuccess,
          onError
        );
      } catch (error) {
        Notify.create({
          message: error,
          color: "negative",
        });
      }
    },
    async chooseFile() {
      Loading.show();
      await chooser
        .getFile("image/*")
        .then((res) => {
          const bytes = res.dataURI.length * (3 / 4) - 1;
          const mb = bytes / 1e6;
          this.image = {
            url: res.dataURI,
            name: res.name.substring(0, 10) + "..." + res.name.slice(-4),
            size: mb.toFixed(2) + " MB",
            file: res.dataURI,
          };

          Loading.hide();
        })
        .catch((err) => {
          Loading.hide();
          Notify.create({
            message: JSON.stringify(err),
            color: "negative",
          });
        });
    },
    async reduceImage(file) {
      return new Promise((resolve, reject) => {
        const $canvas = document.createElement("canvas");
        const imagen = new Image();
        imagen.onload = () => {
          $canvas.width = imagen.width;
          $canvas.height = imagen.height;
          $canvas.getContext("2d").drawImage(imagen, 0, 0);
          $canvas.toBlob(
            (blob) => {
              if (blob === null) {
                return reject(blob);
              } else {
                resolve(blob);
              }
            },
            "image/jpeg",
            this.percent / 100
          );
        };
        imagen.src = file;
      });
    },
  },
  async mounted() {
    // this is the complete list of currently supported params you can pass to the plugin (all optional)
  },
});
</script>
