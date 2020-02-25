<template>
  <nav class="deep-purple darken-1">
    <div class="container">
      <router-link :to="{name:'Home'}" class="brand-logo left">Doğa Klanı</router-link>
      <ul class="right">
        <li v-if="!user">
          <router-link :to="{name:'Signup'}">Signup</router-link>
        </li>
        <li v-if="!user">
          <router-link :to="{name:'Login'}">Login</router-link>
        </li>
        <li v-if="user">
          <router-link :to="{ name: 'ViewProfile', params: { id: user_id }}">{{user.email }}</router-link>
        </li>
        <li v-if="user">
          <a @click="logout">Logout</a>
        </li>
      </ul>
    </div>
  </nav>
</template>

<script>
import firebase from "firebase";
import db from "@/firebase/init";
export default {
  name: "Navbar",
  data() {
    return {
      user: null,
      user_id: null
    };
  },
  methods: {
    logout() {
      firebase
        .auth()
        .signOut()
        .then(() => {
          this.$router.push({ name: "Login" });
        });
    },
    getCurrentUserId() {
      db.collection("users")
        .where("user_id", "==", firebase.auth().currentUser.uid)
        .get()
        .then(snapshot => {
          snapshot.forEach(doc => {
            this.user_id = doc.id;
          });
        });
    }
  },

  created() {
    firebase.auth().onAuthStateChanged(user => {
      if (user) {
        this.user = user;
        this.getCurrentUserId()
      } else {
        this.user = null;
      }
    });
  }
};
</script>
<style >
</style>