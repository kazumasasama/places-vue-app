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
        <button>Update</button>
        <button class="delete-btn">Delete</button>
        <button>Close</button>
      </form>
    </dialog>
  </div>
</template>

<script>
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
    }
  }
}
</script>

<style>
.delete-btn {
  color: red;
}

#map {
  height: 180px;
}
</style>