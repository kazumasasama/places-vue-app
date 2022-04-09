<template>
  <div class="home">
    <div v-for="place in places" :key="place.id" @click="showPlace(place)">
      <hr />
      <p>ID {{ place.id }}</p>
      <h3>{{ place.name }}</h3>
    </div>

    <dialog id="place-dialog">
      <form method="dialog">
        <p>{{ place }}</p>
        <div id="map"></div>
        <button>Update</button>
        <button class="delete-btn">Delete</button>
        <button>Close</button>
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
      place: {
        name: "",
        address: "",
      },
      places: [],
      map: {},
    }
  },
  created() {
    this.indexPlaces();
  },
  methods: {
    indexPlaces() {
      axios.get("places")
        .then((res) => {
          this.places = res.data;
        })
    },
    showPlace(place) {
      this.place = place
      document.getElementById("place-dialog").showModal();

      var map = L.map('map').setView([38.508106, 134.930239], 13);

      L.tileLayer('https://cyberjapandata.gsi.go.jp/xyz/pale/{z}/{x}/{y}.png', {
        attribution: '© 国土地理院'
      }).addTo(map);

      //マーカー表示の記述
      L.marker([34.7019620, 135.509686]).addTo(map)
        .bindPopup('ここにいます')
        .openPopup();
    }
  }
}
</script>

<style>
.delete-btn {
  color: red;
}

#map {
  height: 300px;
  width: 100vw;
}
</style>