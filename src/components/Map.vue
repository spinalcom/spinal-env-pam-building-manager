<!--
Copyright 2022 SpinalCom - www.spinalcom.com

This file is part of SpinalCore.

Please read all of the following terms and conditions
of the Free Software license Agreement ("Agreement")
carefully.

This Agreement is a legally binding contract between
the Licensee (as defined below) and SpinalCom that
sets forth the terms and conditions that govern your
use of the Program. By installing and/or using the
Program, you agree to abide by all the terms and
conditions stated or referenced herein.

If you do not agree to abide by these terms and
conditions, do not demonstrate your acceptance and do
not install or use the Program.
You should have received a copy of the license along
with this file. If not, see
<http://resources.spinalcom.com/licenses.pdf>.
-->

<template>
  <div class="mapContent">
    <MglMap ref="mglMapRef"
            :accessToken="accessToken"
            :mapStyle.sync="mapStyle"
            @load="onMapLoaded"
            :minZoom="0"
            :maxZoom="24">
      <MglMarker v-for="building in buildings"
                 :key="building.id"
                 :color="building.color"
                 :coordinates="getCoordinates(building)"
                 @click="markerOnClick">
        <MglPopup :coordinates="getCoordinates(building)"
                  anchor="top"
                  :closeButton="false">

          <div class="popupContent">
            <div class="name"
                 style="fontWeight: bold">{{building.name}}</div>
            <div class="address">{{building.address}}</div>
          </div>

        </MglPopup>
      </MglMarker>
      <MglNavigationControl position="top-right" />
      <MglGeolocateControl :showUserLocation="true"
                           position="top-right" />

    </MglMap>
  </div>

</template>

<script>
import Vue from "vue";
import Mapbox from "mapbox-gl";
import {
  MglMap,
  MglNavigationControl,
  MglGeolocateControl,
  MglMarker,
  MglAttributionControl,
  MglFullscreenControl,
  MglScaleControl,
  MglPopup,
} from "vue-mapbox";
import { mapMutations } from "vuex";
const mapStyle = require("./mapStyle.json");

export default Vue.extend({
  name: "MapComponent",
  props: { buildings: { default: () => [] } },
  components: {
    MglMap,
    MglNavigationControl,
    MglGeolocateControl,
    MglMarker,
    MglAttributionControl,
    MglFullscreenControl,
    MglScaleControl,
    MglPopup,
  },
  data() {
    return {
      username: "moussabaradji",
      styleId: "cl3stdopz000q15mjloidgds2",
      accessToken:
        "pk.eyJ1IjoibW91c3NhYmFyYWRqaSIsImEiOiJjbDNzaWY2bWEwMGx3M25xc2JpanA0d2E4In0.xWaNYzLeHabH4W-aF0qgpg", // your access token. Needed if you using Mapbox maps
      mapStyle: mapStyle,
    };
  },
  created() {
    this.mapbox = Mapbox;
  },
  // beforeMount() {
  //   this.mapbox = Mapbox;
  //   const url = `https://api.mapbox.com/styles/v1/${this.username}/${this.styleId}?access_token=${this.accessToken}`;

  //   axios
  //     .get(url)
  //     .then((result) => {
  //       this.mapStyle = result.data;
  //     })
  //     .catch((err) => {
  //       console.error(err);
  //     });
  // },
  methods: {
    ...mapMutations("mapStore", ["setComponent"]),
    async onMapLoaded(event) {
      // in component
      // or just to store if you want have access from other components
      // this.$store.map = event.map;

      this.setComponent(event.component);
      const asyncActions = event.component.actions;

      // const newParams = await asyncActions.flyTo({
      //   center: [30, 30],
      //   zoom: 9,
      //   speed: 1,
      // });
      // console.log(newParams);
    },

    getCoordinates(building) {
      const { lat, lng } = building.location;
      return [lng, lat];
    },

    markerOnClick(res) {
      console.log("res", res);
    },
  },
});
</script>

<style scoped>
.mapContent {
  width: 100%;
  height: 100%;
}
</style>