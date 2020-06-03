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
      </section>
    </aside>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      usernames: [],
      newUser: ""
    };
  },
  mounted() {
    if (localStorage.getItem("saved-users").length > 0) {
      this.usernames = JSON.parse(localStorage.getItem("saved-users"));
    }
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
    removeUser: function(user){
      this.usernames.splice(this.usernames.indexOf(user), 1);
      localStorage.setItem("saved-users", JSON.stringify(this.usernames));
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
