<template>
  <div id="app">
    <aside>
      <!--fixed content-->
      <picture>
        <span class="eye"></span>
        <img src="./assets/baselogo.svg" alt="Hush Logo" />
      </picture>
      <div class="divider" />
      <h1>Follow tweets of your favourite people on internet without any noise.</h1>
      <button class="primary" @click="navState='visible'">Edit List</button>
      <div class="divider" />
      <span>
        <a href="https://twitter.com/lilwonderspeaks" target="_blank" rel="noreferrer noopener">
          <img src="./assets/twitter.svg" alt="Twitter Link" />Contact
        </a>&nbsp;<a
          href="https://github.com/thelittlewonder/hush"
          target="_blank"
          rel="noreferrer noopener"
        >GitHub</a>
      </span>
    </aside>
    <!--sliding content-->
    <nav :class="navState">
      <div class="header">
        <h2>Following List</h2>
        <label for="save"></label>
        <input
          type="image"
          @click="saveList()"
          src="./src/assets/close-l.svg"
          alt="close panel"
          id="save"
        />
      </div>
      <div class="divider" />
      <form @submit.prevent="addUser()">
        <label for="username">Enter Twitter username</label>
        <span>
          <input
            required
            autofocus
            type="text"
            id="username"
            placeholder="eg. naval"
            v-model="newUser"
          />
          <button type="submit">→</button>
        </span>
      </form>

      <section>
        <ul>
          <li v-for="user in usernames" :key="user.id">
            <span>
              <h3>{{user.username}}</h3>
              <label for="close"></label>
              <input
                type="image"
                src="./src/assets/remove.svg"
                alt="delete user from list"
                id="close"
                @click="removeUser(user)"
              />
            </span>
          </li>
        </ul>
      </section>
      <p>Changes to this list are saved locally so that you can create a personalised list for yourself.</p>
    </nav>
    <main>
      <transition name="fade">
        <section v-if="!loader && !empty">
          <article v-for="tweet in tweetData" :key="tweet.username">
            <p>{{tweet.content}}</p>
            <span class="info">
              <span class="meta">@{{tweet.author}} • {{tweet.date}}</span>
              <a :href="tweet.url" target="_blank" rel="noreferrer noopener">↗</a>
            </span>
            <div class="divider" />
          </article>
        </section>
      </transition>
      <transition name="fade">
        <div class="emp-st" v-if="empty">
          <h3>You are not following anyone.</h3>
          <p>
            Please
            <span @click="navState='visible'">follow some people</span> to generate your feed
          </p>
        </div>
      </transition>
      <transition name="fade">
        <div class="loader" v-if="loader">
          <span class="ld-text"></span>
        </div>
      </transition>
    </main>
  </div>
</template>

<script>
import "reset-css";
export default {
  name: "app",
  data() {
    return {
      usernames: [
        {
          id: "001",
          username: "shaneaparrish"
        },
        {
          id: "002",
          username: "jamesclear"
        },
        {
          id: "003",
          username: "orangebook_"
        }
      ],
      newUser: "",
      tweetData: "",
      didUpdate: false,
      empty: false,
      loader: true,
      navState: "hidden"
    };
  },
  mounted() {
    let vm = this;
    if (localStorage.getItem("saved-users") == null) {
      localStorage.setItem("saved-users", "");
    }
    if (localStorage.getItem("saved-users").length > 2) {
      vm.empty = false;
      this.usernames = JSON.parse(localStorage.getItem("saved-users"));
    }
    vm.fetchTweets();
  },
  methods: {
    addUser: function(event) {
      let temp = {
        username: this.newUser,
        id:
          Math.random()
            .toString(36)
            .substring(2, 4) +
          Math.random()
            .toString(36)
            .substring(2, 6)
      };
      this.usernames.push(temp);
      localStorage.setItem("saved-users", JSON.stringify(this.usernames));
      this.newUser = "";
      this.didUpdate = true;
    },
    removeUser: function(user) {
      this.usernames.splice(this.usernames.indexOf(user), 1);
      localStorage.setItem("saved-users", JSON.stringify(this.usernames));
      this.didUpdate = true;
    },
    fetchTweets: function() {
      let userlist = "";
      let vm = this;
      this.loader = true;
      for (let i in vm.usernames) {
        userlist = userlist + vm.usernames[i].username + ",";
      }
      userlist = userlist.slice(0, -1);
      let reqUrl = "https://hshapi.herokuapp.com/tweets?u=" + userlist;
      if (userlist.length > 0) {
        fetch(reqUrl)
          .then(response => response.json())
          .then(json => {
            vm.tweetData = json;
            vm.loader = false;
            this.didUpdate = false;
          });
      }
    },
    saveList: function() {
      let vm = this;
      if (this.didUpdate) {
        if (localStorage.getItem("saved-users").length > 2) {
          vm.empty = false;
          vm.fetchTweets();
        } else {
          vm.empty = true;
          vm.loader = false;
        }
      }
      vm.navState = "hidden";
    }
  }
};
</script>

<style lang="scss">
@font-face {
  font-family: "JetBrains Mono";
  font-style: normal;
  font-weight: normal;
  font-display: swap;
  src: url("assets/fonts/JetBrainsMono-Regular.woff2") format("woff2");
}

@font-face {
  font-family: "JetBrains Mono";
  font-style: normal;
  font-weight: 500;
  font-display: swap;
  src: url("assets/fonts/JetBrainsMono-Medium.woff2") format("woff2");
}

body {
  background-color: #1e272e;
  color: #d1c1a8;
  font-family: "JetBrains Mono";
}

.divider {
  height: 0px;
  width: 20px;
  opacity: 0.2;
  border: 1px solid #d1c1a8;
}

@media all and (min-width: 800px) {
  #app {
    display: flex;
    flex-direction: row;
    padding: 48px 96px;
    main {
      margin-left: 120px;
    }
    aside {
      max-width: 280px;
      max-height: 250px;
      overflow: auto;
      position: -webkit-sticky;
      position: sticky;
      top: 48px;
    }

    nav {
      width: 375px;
    }
  }
}

@media all and (min-width: 500px) and (max-width: 799px) {
  #app {
    display: flex;
    flex-direction: column;
    margin: 0 auto;
    padding: 48px;
    main {
      margin-top: 48px;
    }
    aside {
      max-width: 280px;
    }
    nav {
      width: 375px;
    }
  }
}

@media all and (max-width: 499px) {
  #app {
    display: flex;
    flex-direction: column;
    padding: 32px 24px;
    main {
      margin-top: 56px;
    }
    nav {
      max-width: 100%;
    }
  }
}

/*aside*/

aside {
  picture {
    position: relative;
    .eye {
      background-color: #222b33;
      position: absolute;
      height: 2.5px;
      width: 2.5px;
      border-radius: 50%;
      top: -6px;
      left: 6px;
      animation: blink 3s infinite;
    }
    img {
      height: 24px;
    }
  }
  .divider {
    margin: 1em 0;
  }
  h1 {
    font-style: normal;
    font-weight: normal;
    font-size: 0.75em;
    line-height: 1.67em;
    opacity: 0.85;
    margin-bottom: 16px;
  }
  .primary {
    padding: 6px 12px;
    background: #d1c1a8;
    border-radius: 1px;
    font-family: "JetBrains Mono";
    border: none;
    font-style: normal;
    font-weight: 500;
    font-size: 12px;
    line-height: 15px;
    color: #222b33;
    opacity: 0.85;
    margin-bottom: 8px;
    cursor: pointer;
    &:focus {
      outline: none;
      border: none;
    }
  }
  a {
    font-style: normal;
    font-weight: normal;
    font-size: 10px;
    line-height: 12px;
    color: #d1c1a8;
    opacity: 0.5;
    text-decoration: none;
    transition: opacity 0.3s ease-in-out;
    img {
      margin-right: 6px;
    }
    &:hover {
      opacity: 0.8;
    }
  }
}

/*nav*/
nav {
  display: flex;
  flex-direction: column;
  padding: 32px;
  background: #192127;
  overflow: scroll;
  position: fixed;
  bottom: 0;
  transition: left 0.75s ease-in-out;
  height: calc(100% - 64px);
  z-index: 999;
  .header {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    h2 {
      font-weight: 500;
      font-size: 1em;
      line-height: 1.5em;
    }
    input {
      height: 24px;
      width: 24px;
      transition: all 0.3s ease-in-out;
      &:hover {
        transform: scale(1.05);
      }
      &:focus {
        outline: none;
      }
    }
  }
  .divider {
    margin-top: 12px;
  }
  p {
    margin-top: auto;
    font-style: normal;
    font-weight: normal;
    font-size: 10px;
    line-height: 20px;
    color: #d1c1a8;
    opacity: 0.5;
  }
  form {
    display: flex;
    flex-direction: column;
    margin: 40px 0;
    label {
      font-style: normal;
      font-weight: 500;
      font-size: 0.875em;
      line-height: 1.2em;
      color: #d1c1a8;
      opacity: 0.85;
    }
    span {
      display: flex;
      flex-direction: row;
      margin-top: 12px;
      input {
        background: #28343e;
        border-radius: 2px 0px 0px 2px;
        padding: 7px 10px;
        font-size: 0.75em;
        font-family: "JetBrains Mono";
        border: 1px solid #28343e;
        color: #fff;
        transition: all 0.3s ease-in-out;
        &:focus {
          border: 1px solid rgba(209, 193, 168, 0.8);
          outline: none;
          background: #192127;
        }
      }
      button {
        font-family: "JetBrains Mono";
        font-size: 1em;
        line-height: 1.5em;
        padding: 0 16px;
        color: #192127;
        background: #d1c1a8;
        border-radius: 0px 2px 2px 0px;
        border: none;
        cursor: pointer;
        &:focus {
          outline: none;
        }
      }
    }
  }
}

/*Nav Bar States*/
.hidden {
  left: -500px;
}

.visible {
  left: -0px;
}

section {
  ul {
    list-style-type: square;
    margin-left: 1em;
    li {
      display: list-item;
      padding: 10px 0;
      span {
        display: flex;
        align-items: center;
        justify-content: space-between;
        &:hover > input {
          opacity: 1;
        }
        h3 {
          font-style: normal;
          font-weight: normal;
          font-size: 13px;
          line-height: 16px;
          letter-spacing: 0.01em;
          color: #d1c1a8;
          opacity: 0.85;
          cursor: default;
        }
        input {
          height: 20px;
          width: 20px;
          padding: 4px;
          opacity: 0;
          transition: all 0.3s ease-in-out;
          &:focus {
            outline: none;
          }
          &:hover {
            transform: scale(1.05);
          }
        }
        @media (hover: none) {
          input{
            opacity: 1;
          }
        }
      }
    }
  }
}

main {
  grid-column: 2 / span 2;
  article {
    max-width: 450px;
    p {
      font-weight: 500;
      font-size: 0.875em;
      line-height: 1.7em;
      text-align: justify;
      color: #d1c1a8;
    }
    .info {
      margin-top: 16px;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      .meta {
        font-style: normal;
        font-weight: normal;
        font-size: 11px;
        color: #d1c1a8;
        opacity: 0.75;
      }
      a {
        font-style: normal;
        font-weight: normal;
        font-size: 11px;
        color: #d1c1a8;
        opacity: 0.75;
        padding: 4px;
        text-decoration: none;
      }
    }
    .divider {
      opacity: 0.2;
      border: 1px solid #d1c1a8;
      width: 20px;
      margin: 40px 0;
    }
  }
}

.loader {
  .ld-text {
    font-size: 0.875em;
    &::before {
      content: "Fetching Tweets ";
    }
    &::after {
      content: "\00a0\00a0\00a0";
      animation: progress-ellipsis 3s infinite;
    }
  }
}

@keyframes progress-ellipsis {
  0% {
    content: "\00a0\00a0\00a0";
  }
  30% {
    content: ".\00a0\00a0";
  }
  60% {
    content: ". .\00a0";
  }
  90% {
    content: ". . .";
  }
}

.emp-st {
  color: #fff;
  h3 {
    font-size: 0.875em;
    opacity: 0.6;
  }
  p {
    font-size: 0.75em;
    margin-top: 12px;
    opacity: 0.4;
    span {
      cursor: pointer;
      color: #d1c1a8;
      text-decoration: underline dotted;
    }
  }
}
@keyframes blink {
  0%,
  100% {
    transform: scale(1, 0.02);
  }

  10%,
  90% {
    transform: scale(1, 1);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
