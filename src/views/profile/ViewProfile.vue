<template>
  <div class="view-profile container">

    <div class="card" v-if="profile">
      <h2 class="deep-purple-text center">{{ profile.alias }} ' Wall</h2>
      <ul class="comments collection">
        <li v-for="(comment,index) in comments" :key="index">
          <div class="deep-purple-text">
              <router-link :to="{ name: 'ViewProfile', params: { id: comment.from_slug }}">{{comment.from }}</router-link>
          </div>
          <div class="grey-text text-darken-2">{{comment.content}}</div>
          <div class="grey-text lighten-1 time">{{comment.time}}</div>
        </li>
      </ul>
      <form @submit.prevent="addComment">
        <div class="field">
          <label for="comment">Add a comment</label>
          <input type="text" name="comment" v-model="newComment" />
          <i class="material-icons deep-purple-text" @click="addComment">send</i>
          <p v-if="feedback" class="red-text center">{{feedback}}</p>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import db from "@/firebase/init";
import firebase from "firebase";
import moment from "moment";
export default {
  name: "ViewProfile",
  data() {
    return {
      profile: null, 
      newComment: null,
      feedback: null,
      user: null, 
      users: null,
      comments: [],
    };
  },
  created() {
    this.users = db.collection("users");
    
    this.getCurrentUser();

    this.getProfileData();

    this.getComments();
  },
  methods: {
    addComment() {
      if (this.newComment) {
        this.feedback = null;
        db.collection("comments")
          .add({
            to: this.$route.params.id,
            from: this.user.alias,
            from_slug:this.user.id,
            content: this.newComment,
            time: Date.now()
          })
          .then(() => {
            this.newComment = null;
          });
      } else {
        this.feedback = "You must enter a comment to add it ";
      }
    },
    getProfileData() {
      this.users
        .doc(this.$route.params.id)
        .get()
        .then(user => {
          this.profile = user.data();
        });
    },
    getCurrentUser() {
      this.users
        .where("user_id", "==", firebase.auth().currentUser.uid)
        .get()
        .then(snapshot => {
          snapshot.forEach(doc => {
            this.user = doc.data()
            this.user.id = doc.id
          });
        });
      
    },
    getComments() {
      db.collection("comments")
        .where("to", "==", this.$route.params.id)
        .onSnapshot(snapshot => {
          snapshot.docChanges().forEach(change => {
            if (change.type == "added") {
              this.comments.push({
                from: change.doc.data().from,
                from_slug:change.doc.data().from_slug,
                content: change.doc.data().content,
                time: moment(change.doc.data().time).format("lll")
              });
            }
          });
        });
    },


  },
   beforeRouteEnter (to, from, next) {
       next()
  },

  watch: {
    $route () {
        this.comments=[]
        this.getProfileData()
        this.getComments()
    }
  }
};
</script>
<style>
.view-profile .card {
  padding: 20px;
  margin-top: 60px;
}
.view-profile h2 {
  font-size: 2em;
  margin-top: 0;
}
.view-profile li {
  padding: 10px;
  border-bottom: 1px solid #eee;
}
.view-profile .material-icons {
  position: absolute;
  right: 40px;
  cursor: pointer;
}
.view-profile .time {
  font-size: 0.7em;
}
</style>