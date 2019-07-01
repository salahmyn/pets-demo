<template>
  <v-app>
    <v-content>
      <v-layout
        align-center
        justify-center
        row
        fill-height
        v-show="!loaded"
        transition="fade-transition"
      >
        <v-progress-circular indeterminate color="blue"></v-progress-circular>
      </v-layout>

      <v-card v-show="loaded" transition="slide-x-transition">
        <v-toolbar color="cyan" dark>
          <v-toolbar-title>Pets Directory</v-toolbar-title>

          <v-spacer></v-spacer>

          <v-btn icon>
            <v-icon>search</v-icon>
          </v-btn>
        </v-toolbar>
        <div class="filters-box">
          <v-radio-group v-model="petType" row>
            <label for>Pet Type:</label>
            <v-radio v-for="n in petTypes" :key="n" :label="n" :value="n"></v-radio>
          </v-radio-group>
          <v-radio-group v-model="ownerGender" row>
            <label for>Owner Gender:</label>
            <v-radio key="all" label="All" value></v-radio>
            <v-radio key="male" label="Male" value="Male"></v-radio>
            <v-radio key="female" label="Female" value="Female"></v-radio>
          </v-radio-group>
        </div>

        <v-list two-line>
          <template v-for="(pet, index) in filteredPets">
            <v-list-tile :key="index" avatar>
              <v-list-tile-avatar>
                <img :src="getAvatar(pet.type)" />
              </v-list-tile-avatar>

              <v-list-tile-content>
                <v-list-tile-title>
                  {{ pet.name }}
                  <small class="pet-type">{{ pet.type }}</small>
                </v-list-tile-title>
                <v-list-tile-sub-title>Owned By: {{ pet.owner_name }}, {{ pet.owner_gender }}, {{ pet.owner_age }} years old</v-list-tile-sub-title>
              </v-list-tile-content>
            </v-list-tile>
          </template>
        </v-list>
      </v-card>
    </v-content>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  components: {},
  data() {
    return {
      owners: [],
      petType: "Cat",
      ownerGender: null,
      loaded: false
    };
  },
  computed: {
    sortedPets() {
      let all_pets = [];
      for (
        let owner_index = 0;
        owner_index < this.owners.length;
        owner_index++
      ) {
        const owner = this.owners[owner_index];
        if (owner.pets) {
          const pets = owner.pets.map(pet => {
            return {
              owner_id: owner_index,
              owner_name: owner.name,
              owner_gender: owner.gender,
              owner_age: owner.age,
              ...pet
            };
          });
          all_pets = all_pets.concat(pets);
        }
      }
      return all_pets.sort((a, b) => a.name.localeCompare(b.name));
    },
    petTypes() {
      return [...new Set(this.sortedPets.map(p => p.type))];
    },
    filteredPets() {
      let self = this;
      return this.sortedPets.filter(pet => {
        if (self.ownerGender) {
          return (
            pet.type == self.petType && pet.owner_gender == self.ownerGender
          );
        }
        return pet.type == self.petType;
      });
    }
  },
  methods: {
    getAvatar(type) {
      if (type == "Cat") {
        return require("./assets/cat.png");
      }
      if (type == "Dog") {
        return require("./assets/dog.png");
      }
      return require("./assets/unkown.png");
    }
  },
  created() {
    axios
      .get("http://5c92dbfae7b1a00014078e61.mockapi.io/owners")
      .then(res => {
        this.owners = res.data;
        this.loaded = true;
      })
      .catch(error => {
        // handel error
        console.log(error);
      });
  }
};
</script>
<style>
.pet-type {
  vertical-align: text-bottom;
  font-size: 10px;
  color: #828282;
}
.filters-box {
  padding: 20px 20px 0 20px;
  background: aliceblue;
}
</style>