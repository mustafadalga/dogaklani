<template>
  <div class="map">
    <div class="google-map" id="map"></div>

  </div>
</template>


<script>
import firebase from "firebase";
import db from "@/firebase/init";
export default {
  name: "Home",
  data() {
    return {
      lat: 30,
      lng: 35,
      map: null,
      user: null
    };
  },
  methods: {
    renderMap() {
      // eslint-disable-next-line no-undef
      this.map = new google.maps.Map(document.getElementById("map"), {
        center: { lat: this.lat, lng: this.lng },
        zoom: 6,
        maxZoom: 15,
        minZoom: 3
      });

      this.renderUsersLocation();
    },
    renderUsersLocation() {
      db.collection("users")
        .get()
        .then(users => {
          users.docs.forEach(doc => {
            let data = doc.data();
            if (data.geolocation) {
              // eslint-disable-next-line no-undef
              let marker = new google.maps.Marker({
                position: {
                  lat: data.geolocation.lat,
                  lng: data.geolocation.lng
                },
                map: this.map
              });
              marker.addListener("click", () => {
                this.$router.push({
                  name: "ViewProfile",
                  params: { id: doc.id }
                });
              });
            }
          });
        });
    },
    getUserLocation() {
      this.user = firebase.auth().currentUser;

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          pos => {
            (this.lat = pos.coords.latitude), (this.lng = pos.coords.longitude);

            this.updateUserLocation();
          },
          error => {
              console.log(error);
            this.renderMap();
          },
          { maxium: 60000, timeout: 3000 }
        );
      } else {
        this.renderMap();
      }
    },
    updateUserLocation() {
      db.collection("users")
        .where("user_id", "==", this.user.uid)
        .get()
        .then(snapshot => {
          snapshot.forEach(doc => {
            db.collection("users")
              .doc(doc.id)
              .update({
                geolocation: {
                  lat: this.lat,
                  lng: this.lng
                }
              });
          });
        })
        .then(() => {
          this.renderMap();
        });
    }
  },
  mounted() {
    this.getUserLocation();
  }
};
</script>


<style>
.google-map {
  width: 100%;
  height: 100%;
  margin: 0 auto;
  background: #fff;
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>