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
  <v-container class="homeContainer"
               fluid>
    <div class="rowContainer">
      <div class="mapContainer">
        <map-view :buildings="searchedData"></map-view>
      </div>

      <div class="appContainer"
           v-if="state === STATES.normal">

        <div class="header">
          <div class="header_col left">
            <v-text-field label="Search"
                          dark
                          append-icon="mdi-magnify"
                          v-model="searchText"></v-text-field>
          </div>

          <div class="header_col right">
            <button class="addButton"
                    @click="addBuilding">Add Building</button>
          </div>

        </div>
        <div class="content">
          <patrimoine-card v-for="building in searchedData"
                           :key="building.id"
                           :data="building"
                           @viewOnMap="viewOnMap"
                           @edit="editBuilding"
                           @delete="deleteBuilding"></patrimoine-card>
        </div>
      </div>

      <div class="appContainer"
           v-else-if="state === STATES.creating || state === STATES.editing">
        <div class="header creationHeader">

          <v-btn outlined
                 color="#ffffff"
                 @click="cancelCreation">
            <v-icon left>
              mdi-arrow-left
            </v-icon>
            BACK
          </v-btn>

          <div class="text-h5">
            {{state === STATES.editing ? "Edit Building" : "Add Building"}}
          </div>
          <div class="text-h5"></div>
        </div>
        <div class="content">
          <add-patrimoine-form :edit="state === STATES.editing"
                               :buildingToEdit="buildingToEdit"
                               @geoLocate="geoLocate"
                               @cancel="cancelCreation"
                               @submit="addbuildingToPatrimoine">
          </add-patrimoine-form>
        </div>

      </div>

      <div class="appContainer"
           v-else>
        <div class="content loadingContent">
          <div v-if="state === STATES.loading">loading...</div>
          <div v-else-if="state === STATES.error">
            Error
            <!-- <v-icon color="error"
                    x-large>mdi-alert-outline</v-icon> -->
          </div>
        </div>

      </div>
    </div>
  </v-container>
</template>

<script>
import Vue from "vue";
import MapView from "@/components/Map.vue";
import ListView from "@/components/List.vue";
import { mapActions, mapState } from "vuex";
import PatrimoineCard from "../components/patrimoine-card.vue";
import AddPatrimoineForm from "../components/addBuilding.vue";
import Swal from "sweetalert2";
import * as lodash from "lodash";

export default Vue.extend({
  name: "Home",
  components: {
    MapView,
    ListView,
    PatrimoineCard,
    AddPatrimoineForm,
  },
  data() {
    this.STATES = Object.freeze({
      normal: 0,
      creating: 1,
      editing: 2,
      loading: 3,
      error: 4,
    });
    return {
      state: this.STATES.normal,
      buildingToEdit: null,
      searchText: "",
      searchedData: [],
    };
  },
  created() {
    this.searchAndFilterTable = lodash.debounce(this.filterBuilding, 500);
  },
  mounted() {
    this.initializeData();
  },
  methods: {
    ...mapActions("digitalTwinStore", [
      "getAllDigitalTwin",
      "deleteDigitalTwin",
      "addDigitalTwin",
      "editDigitaltwin",
    ]),

    async initializeData() {
      try {
        this.state = this.STATES.loading;
        await this.getAllDigitalTwin();
        this.state = this.STATES.normal;
      } catch (error) {
        console.error(error);
        this.state = this.STATES.error;
      }
    },

    addBuilding() {
      this.state = this.STATES.creating;
    },

    cancelCreation() {
      this.state = this.STATES.normal;
    },

    async addbuildingToPatrimoine({ data, create }) {
      this.state = this.STATES.loading;
      try {
        if (create) await this.addDigitalTwin(data);
        else await this.editDigitaltwin({ id: data.id, data });
        this.state = this.STATES.normal;
      } catch (error) {
        this.state = this.STATES.error;
      }
    },

    viewOnMap(item) {
      const { lat, lng } = item.location;
      this.component.actions.easeTo({
        zoom: 18,
        center: { lng, lat },
        duration: 500,
      });
    },

    editBuilding(item) {
      this.state = this.STATES.editing;
      this.buildingToEdit = item;
    },

    deleteBuilding(item) {
      Swal.fire({
        title: "Are you sure ?",
        text: "You won't be able to revert this!",
        type: "warning",
        showCancelButton: true,
        confirmButtonClass: "confirmBtn",
        cancelButtonClass: "cancelBtn",
        confirmButtonText: "Yes",
        cancelButtonText: "No",
      }).then(async (result) => {
        if (result.isConfirmed) await this.deleteDigitalTwin(item.id);
      });
    },

    geoLocate() {
      // mapbox.GeolocateControl
      const geoControl = this.component.map._controls.find(
        (el) => el.constructor.name === "GeolocateControl"
      );

      if (geoControl) {
        geoControl.trigger();
      }
    },

    // changeSearchText: lodash.debounce((value) => {
    //   // this.searchText = value;
    //   console.log();
    // }, 400),

    filterBuilding() {
      this.searchedData = !!this.searchText
        ? this.buildings.filter((el) =>
            el.name.toLowerCase().includes(this.searchText.toLowerCase())
          )
        : this.buildings;
    },
  },
  computed: {
    ...mapState("digitalTwinStore", ["buildings"]),
    ...mapState("mapStore", ["component"]),
  },
  watch: {
    buildings() {
      this.searchAndFilterTable();
    },
    searchText() {
      this.searchAndFilterTable();
    },
  },
});
</script>

<style scoped>
.homeContainer {
  width: 100%;
  height: calc(100%);
  padding: 0px;
  overflow: hidden;
}
.homeContainer .rowContainer {
  width: 100%;
  height: 100%;
  display: flex;
  margin: 0px !important;
}

.homeContainer .rowContainer .mapContainer,
.homeContainer .rowContainer .appContainer {
  height: calc(100% - 10px);
  padding: 0px !important;
  overflow: hidden;
}

.homeContainer .rowContainer .mapContainer {
  flex: 0 0 65%;
}

.homeContainer .rowContainer .appContainer {
  flex: 0 0 35%;
}

.homeContainer .mapContainer {
  width: 100%;
  height: 100%;
}

.homeContainer .rowContainer .appContainer {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 5px;
  overflow: hidden;

  /* 
    background: #14202c;
    border-left: 5px solid white; 
  */
}

.homeContainer .appContainer .header {
  width: 98%;
  height: 8vh;
  background: #14202c;
  padding: 10px;
  border-radius: 15px;
  margin-bottom: 10px;
  margin-top: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
}

.homeContainer .appContainer .header .header_col.left {
  flex: 0 0 70%;
}

.homeContainer .appContainer .header .header_col.right {
  flex: 0 0 30%;
  display: flex;
  justify-content: end;
}

.homeContainer .appContainer .header.creationHeader {
  display: flex;
  justify-content: space-between;
}

.homeContainer .appContainer .header .addButton {
  width: 150px;
  height: 40px;
  border: 1px solid #fff;
  border-radius: 10px;
  text-transform: uppercase;
}

.homeContainer .appContainer .content {
  width: 98%;
  height: 90vh;
  padding: 10px 0;
  background: #f1f5f5;
  border-radius: 10px;
  overflow: auto;
}

.homeContainer .appContainer .content.loadingContent {
  height: 98vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>

<style>
.cancelBtn {
  background: #ff5252 !important;
}

.confirmBtn {
  background: #4caf50 !important;
}
</style>