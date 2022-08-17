<template>
  <div class="_container">
    <div class="header">
      <div class="header_col left">
        <v-text-field
          label="Search"
          dark
          append-icon="mdi-magnify"
          v-model="searchText"
        ></v-text-field>
      </div>

      <div class="header_col right">
        <button class="addButton" @click="addBuilding">Add Building</button>
      </div>
    </div>

    <div class="content">
      <patrimoine-card
        v-for="building in buildings"
        :key="building.id"
        :data="building"
        @viewOnMap="viewOnMap"
        @edit="editBuilding"
        @delete="deleteBuilding"
      ></patrimoine-card>
    </div>
  </div>
</template>

<script>
import PatrimoineCard from "../patrimoine-card.vue";

export default {
  name: "BuildingView",
  components: {
    PatrimoineCard,
  },
  props: ["buildings"],
  data() {
    return {
      searchText: "",
    };
  },
  methods: {
    addBuilding() {
      this.$emit("addBuilding");
    },
    viewOnMap(item) {
      this.$emit("viewOnMap", item);
    },
    editBuilding(item) {
      console.log("item",item)
      this.$emit("edit", item);
    },
    deleteBuilding(item) {
      this.$emit("delete", item);
    },
  },
  watch: {
    searchText() {
      this.$emit("filter", this.searchText);
    },
  },
};
</script>

<style scoped>
._container {
  width: 98%;
  height: calc(100%);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

._container .header {
  width: 97%;
  height: 70px;
  background: #14202c;
  padding: 0 10px;
  border-radius: 10px;
  margin-bottom: 10px;
  margin-top: 2px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
}

._container .header .header_col.left {
  flex: 0 0 65%;
}

._container .header .header_col.right {
  flex: 0 0 35%;
  display: flex;
  justify-content: end;
}

._container .content {
  width: 97%;
  height: calc(100% - 90px);
  padding: 10px 0;
  background: #f1f5f5;
  border-radius: 10px;
  overflow: auto;
}

._container .header .addButton {
  width: 150px;
  height: 40px;
  border: 1px solid #fff;
  border-radius: 10px;
  text-transform: uppercase;
}
</style>