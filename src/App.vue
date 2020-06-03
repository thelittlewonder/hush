<template>
  <div id="app">
    <aside>
      <form @submit.prevent="addUser()">
        <label for="username">Follow a user</label>
        <input autofocus type="text" name="username" placeholder="naval" v-model="newUser" />
        <button type="submit">Add</button>
      </form>
      <section>
        <div v-for="user in usernames" :key="user.id">
          <p>{{user.username}}</p>
          <button @click="removeUser(user)">x</button>
        </div>
        <button @click="fetchTweets()">Save</button>
      </section>
    </aside>
    <main v-if="!empty">{{tweetData}}</main>
    <main v-else>Please follow some people</main>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      usernames: [],
      newUser: "",
      tweetData: "",
      empty: false
    };
  },
  mounted() {
    let vm = this;
    if (localStorage.getItem("saved-users").length > 2) {
      this.usernames = JSON.parse(localStorage.getItem("saved-users"));
      vm.fetchTweets()
    }

    this.fetchTweets();
  },
  methods: {
    addUser: function(event) {
      let temp = {
        username: this.newUser,
        id: this.usernames.length + 1
      };
      this.usernames.push(temp);
      localStorage.setItem("saved-users", JSON.stringify(this.usernames));
      this.newUser = "";
    },
    removeUser: function(user) {
      this.usernames.splice(this.usernames.indexOf(user), 1);
      localStorage.setItem("saved-users", JSON.stringify(this.usernames));
    },
    fetchTweets: function() {
      let userlist = "";
      let vm = this;
      for (let i in vm.usernames) {
        userlist = userlist + vm.usernames[i].username + ",";
      }
      userlist = userlist.slice(0, -1);
      let reqUrl = "http://127.0.0.1:5000/tweets?u=" + userlist;
      if (userlist.length > 0) {
        fetch(reqUrl)
          .then(response => response.json())
          .then(json => (vm.tweetData = json));
      } else {
        vm.empty = true
      }
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
