<template>
  <div class="home">
    <div v-for="error in errors" :key="error">
      <p class="error-msg">{{ error }}</p>
    </div>
    <div>
      <button @click="createPlace()">Add place</button>
      <p>
        <input v-model="newPlace.name" type="text" placeholder="Name" />
      </p>
      <p>
        <input v-model="newPlace.address" type="text" placeholder="Address" />
      </p>
    </div>
    <div v-for="place in places" :key="place.id" @click="showPlace(place)">
      <hr />
      <p>ID {{ place.id }}</p>
      <h3>{{ place.name }}</h3>
    </div>

    <dialog id="place-dialog">
      <form method="dialog">
        <p>
          Name
          <input v-model="place.name" type="text" />
        </p>
        <p>
          Address
          <input v-model="place.address" type="text" />
        </p>
        <p>Longitude: {{ place.longitude }} Latitude: {{ place.latitude }}</p>
        <div id="map"></div>
        <button @click="updatePlace(place)">Update</button>
        <button @click="destroyPlace(place)" class="delete-btn">Delete</button>
        <button @click="handleClose()">Close</button>
      </form>
    </dialog>
  </div>
</template>

<script>
import L from "../../node_modules/leaflet/dist/leaflet"
import axios from "axios"
export default {
  name: 'HomeView',
  data() {
    return {
      errors: [],
      place: {
        name: "",
        address: "",
        longitude: null,
        latitude: null,
      },
      places: [],
      map: null,
      newPlace: {},
    }
  },
  created() {
    this.indexPlaces();
  },
  methods: {
    indexPlaces() {
      axios.get("places.json")
        .then((res) => {
          this.places = res.data;
        })
    },
    showPlace(place) {
      this.place = place
      if (!this.map) {
        this.map = new L.map("map").setView([place.latitude, place.longitude], 13);
        L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1,
          accessToken: process.env.VUE_APP_MAPBOX
        }).addTo(this.map);
        this.places.forEach((place) => {
          L.marker([place.latitude, place.longitude]).addTo(this.map)
            .bindPopup(`${place.name}`)
            .openPopup();
        })
      }

      L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: 'mapbox/streets-v11',
        tileSize: 512,
        zoomOffset: -1,
        accessToken: process.env.VUE_APP_MAPBOX
      }).addTo(this.map);

      document.getElementById("place-dialog").showModal();
    },
    createPlace() {
      this.errors = [];
      axios.post("places.json", this.newPlace)
        .then((res) => {
          this.places.push(res.data);
          this.newPlace = {};
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
        })
    },
    updatePlace(place) {
      this.errors = [];
      axios.patch(`places/${place.id}.json`, place)
        .then((res) => {
          var i = this.places.indexOf(res.data)
          this.places[i] = res.data
        })
    },
    destroyPlace(place) {
      this.errors = [];
      axios.delete(`places/${place.id}.json`)
        .then((res) => {
          var i = this.places.indexOf(res.data)
          this.places.splice(i, 1);
          this.place = {};
        }).catch((error) => {
          this.errors = error.response.data.errors;
        })
    },
    handleClose() {
      this.place = {};
      this.map.options.center = [];
    },
  }
}
</script>

<style>
.delete-btn {
  color: red;
}

#map {
  height: 300px;
}
</style>