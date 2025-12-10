<template>
  <v-layout>
    <v-flex xs12 md6 lg4>
      <!-- Personal Access Token Section -->
      <v-card class="mb-4" dark>
        <v-card-title>Personal Access Token</v-card-title>
        <v-card-text>
          <p class="caption mb-3">
            Use this token to authenticate API requests and integrate Scrypted with external applications.
            Keep your token secure - it provides full access to your Scrypted server.
          </p>
          <v-text-field
            v-model="token"
            readonly
            outlined
            dense
            :type="showToken ? 'text' : 'password'"
            label="Token"
            :loading="tokenLoading"
            :append-icon="showToken ? 'visibility' : 'visibility_off'"
            @click:append="showToken = !showToken"
          >
            <template v-slot:append-outer>
              <v-btn
                small
                icon
                @click="copyToken"
                :disabled="!token"
                title="Copy to clipboard"
              >
                <v-icon small>content_copy</v-icon>
              </v-btn>
            </template>
          </v-text-field>
          <v-btn
            small
            outlined
            color="orange"
            @click="regenerateDialog = true"
            :disabled="tokenLoading"
            class="mt-2"
          >
            <v-icon small left>refresh</v-icon>
            Regenerate Token
          </v-btn>
          <div v-if="tokenMessage" class="mt-2 caption" :class="tokenError ? 'error--text' : 'success--text'">
            {{ tokenMessage }}
          </div>
        </v-card-text>
      </v-card>

      <!-- Regenerate Token Confirmation Dialog -->
      <v-dialog v-model="regenerateDialog" width="500">
        <v-card color="orange" dark>
          <v-card-title primary-title>Regenerate Token</v-card-title>
          <v-card-text>
            Are you sure you want to regenerate your personal access token? 
            This will invalidate the current token and any integrations using it will need to be updated.
          </v-card-text>
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn text @click="regenerateDialog = false">Cancel</v-btn>
            <v-btn text @click="doRegenerateToken">Regenerate</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- <GmapMap
        :center="position"
        :zoom="16"
        ref="mapRef"
        style="height: 400px"
        :options="{
          mapTypeControl: false,
          fullscreenControl: false,
        }"
      >
        <GmapMarker :position="position" />
      </GmapMap>

      <v-flex>
        <v-text-field
          ref="locationAutocomplete"
          autocomplete="off"
          outlined
          label="Set Location (Maps, Timezone)"
          placeholder="Set Location"
          v-model="location"
        ></v-text-field>
      </v-flex>

      <v-flex>
        <v-btn color="primary" @click="goLegacy" outlined dark block
          >Legacy Management Console</v-btn
        >
      </v-flex>

      <v-flex>
        <form method="POST" action="/web/component/settings/backup">
          <v-btn color="green" type="submit" outlined dark block
            >Download Backup</v-btn
          >
        </form>
      </v-flex>

      <v-flex>
        <input
          type="file"
          name="file"
          hidden
          ref="restoreInput"
          @change="doRestore"
        />
        <v-btn color="blue" outlined dark block @click="restore"
          >Restore Backup</v-btn
        >
      </v-flex> -->

      <v-dialog v-if="showRestart" v-model="restart" width="500">
        <template v-slot:activator="{ on }">
          <v-flex>
            <v-btn class="mb-2" block color="red" dark v-on="on"
              >Restart Scrypted</v-btn
            >
          </v-flex>
        </template>

        <v-card color="red" dark>
          <v-card-title primary-title>Restart Scrypted</v-card-title>

          <v-card-text
            >Are you sure you want to restart the Scrypted service?</v-card-text
          >

          <v-card-text>{{ restartStatus }}</v-card-text>
          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn text @click="restart = false">Cancel</v-btn>
            <v-btn text @click="doRestart">Restart</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog v-if="showUpdate" v-model="updateAndRestart" width="500">
        <template v-slot:activator="{ on }">
          <v-flex>
            <v-btn block color="red" dark v-on="on"
              >Update and Restart Scrypted</v-btn
            >
          </v-flex>
        </template>

        <v-card color="red" dark>
          <v-card-title primary-title>Restart Scrypted</v-card-title>

          <v-card-text
            >Are you sure you want to restart the Scrypted service?</v-card-text
          >

          <v-card-text>{{ restartStatus }}</v-card-text>
          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn text @click="updateAndRestart = false">Cancel</v-btn>
            <v-btn text @click="doUpdateAndRestart">Restart</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-flex>
  </v-layout>
</template>
<script>
import { getComponentWebPath } from "../helpers";
import axios from "axios";
import throttle from "lodash/throttle";
import qs from "query-string";

export default {
  data() {
    return {
      updateAndRestart: false,
      restart: false,
      restartStatus: undefined,
      showRestart: false,
      showUpdate: false,
      token: '',
      showToken: false,
      tokenLoading: false,
      tokenMessage: '',
      tokenError: false,
      regenerateDialog: false,
    };
  },
  // data() {
  //   return {
  //     restart: false,
  //     restartStatus: undefined,
  //     location: "",
  //     position: {
  //       lat: 0,
  //       lng: 0,
  //     },
  //   };
  // },
  // computed: {
  //   componentWebPath() {
  //     return getComponentWebPath("settings");
  //   },
  // },
  mounted() {
    this.loadEnv();
    this.loadToken();
    // this.$refs.mapRef.$mapPromise.then(() => {
    //   let element = this.$refs.locationAutocomplete.$el;
    //   element = element.querySelector("input");
    //   var autocomplete = new google.maps.places.Autocomplete(element, {
    //     types: ["geocode"],
    //   });
    //   autocomplete.addListener("place_changed", () => {
    //     var place = autocomplete.getPlace();
    //     this.location = place.formatted_address;
    //     this.position.lat = place.geometry.location.lat();
    //     this.position.lng = place.geometry.location.lng();
    //     this.debounceUpdate(
    //       place.geometry.location.lat().toString(),
    //       place.geometry.location.lng().toString()
    //     );
    //   });
    // });
    // axios
    //   .get(`${this.getComponentWebPath("automation")}/settings`)
    //   .then((response) => {
    //     this.location = response.data.location;
    //     this.position.lat = parseFloat(response.data.latitude);
    //     this.position.lng = parseFloat(response.data.longitude);
    //   });
  },
  methods: {
    async loadToken() {
      this.tokenLoading = true;
      this.tokenMessage = '';
      this.tokenError = false;
      try {
        const response = await axios.get('/web/api/token');
        this.token = response.data.token;
      } catch (error) {
        this.tokenMessage = 'Failed to load token';
        this.tokenError = true;
        console.error('Failed to load token:', error);
      } finally {
        this.tokenLoading = false;
      }
    },
    async doRegenerateToken() {
      this.regenerateDialog = false;
      this.tokenLoading = true;
      this.tokenMessage = '';
      this.tokenError = false;
      try {
        const response = await axios.post('/web/api/token/regenerate');
        this.token = response.data.token;
        this.tokenMessage = 'Token regenerated successfully';
        this.tokenError = false;
      } catch (error) {
        this.tokenMessage = 'Failed to regenerate token';
        this.tokenError = true;
        console.error('Failed to regenerate token:', error);
      } finally {
        this.tokenLoading = false;
      }
    },
    copyToken() {
      if (!this.token) return;
      
      // Create a temporary input element to copy the token
      const input = document.createElement('input');
      input.value = this.token;
      document.body.appendChild(input);
      input.select();
      
      try {
        document.execCommand('copy');
        this.tokenMessage = 'Token copied to clipboard';
        this.tokenError = false;
        setTimeout(() => {
          this.tokenMessage = '';
        }, 3000);
      } catch (error) {
        this.tokenMessage = 'Failed to copy token';
        this.tokenError = true;
        console.error('Failed to copy token:', error);
      } finally {
        document.body.removeChild(input);
      }
    },
    async loadEnv() {
      const info = await this.$scrypted.systemManager.getComponent("info");
      const env = await info.getScryptedEnv();
      this.showRestart = !!env.SCRYPTED_CAN_RESTART;
      this.showUpdate = !!(env.SCRYPTED_NPM_SERVE || env.SCRYPTED_GIT_SERVE);
    },
    // getComponentWebPath,
    // debounceUpdate: throttle(function (latitude, longitude) {
    //   axios.post(
    //     `${this.getComponentWebPath("automation")}/`,
    //     qs.stringify({
    //       location: this.location,
    //       latitude,
    //       longitude,
    //     }),
    //     {
    //       "Content-Type": "application/x-www-form-urlencoded",
    //     }
    //   );
    // }, 500),
    // goLegacy() {
    //   window.open("/web/dashboard");
    // },
    async doRestart() {
      this.restartStatus = "Restarting...";
      const serviceControl = await this.$scrypted.systemManager.getComponent(
        "service-control"
      );
      await serviceControl.restart();
    },
    async doUpdateAndRestart() {
      this.restartStatus = "Restarting...";
      const serviceControl = await this.$scrypted.systemManager.getComponent(
        "service-control"
      );
      await serviceControl.update();
    },
    // restore() {
    //   this.$refs.restoreInput.click();
    // },
    // doRestore() {
    //   let formData = new FormData();
    //   formData.append("file", this.$refs.restoreInput.files[0]);
    //   axios
    //     .post("/web/component/settings/restore", formData, {
    //       headers: {
    //         "Content-Type": "multipart/form-data",
    //       },
    //     })
    //     .then(function () {})
    //     .catch(function () {});
    // },
  },
};
</script>
