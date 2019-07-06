<template>
  <div id="userProfile" :style="getProfileStyle()" class="profileScreen">
    <v-scroll-y-transition mode="out-in">
      <div
        v-if="!activeUser"
        class="title grey--text text--lighten-1 font-weight-light"
        style="display: flex; justify-content: center; align-items: center; height: 90%;"
      >No user is selected</div>
      <v-card v-else :key="activeUser.id" class="mx-auto pt-4" flat max-width="400">
        <v-card-text style="display: flex; justify-content: center; flex-wrap: wrap;">
          <v-avatar class="centered" v-if="activeUser.avatar" size="88">
            <v-img :src="activeUser.avatar" class="mb-4"></v-img>
          </v-avatar>
          <h3 class="centered headline my-2">{{ activeUser.firstName }}</h3>
          <div class="centered blue--text mb-2">{{ activeUser.email }}</div>
          <div class="centered">
            <v-icon color="blue">mdi-twitter</v-icon>

            <div
              class="blue--text subheading font-weight-bold"
            >{{ activeUser.twitter.replace('@','') }}</div>
          </div>
        </v-card-text>
        <v-divider></v-divider>
        <v-layout tag="v-card-text" text-xs-left wrap>
          <v-flex tag="strong" xs5 text-xs-right mr-3 mb-2>Phone:</v-flex>
          <v-flex>{{ activeUser.phone }}</v-flex>
        </v-layout>
      </v-card>
    </v-scroll-y-transition>
  </div>
</template>

<script>
import leylo from "@/leylo";

export default {
  name: "userprofile",
  computed: {
    app() {
      return this.$root.$children[0];
    },
    userList() {
      return this.app.main ? this.app.main.userList : null;
    },
    canRender() {
      return (
        document.getElementById("userList") &&
        document.getElementsByClassName("splitter-pane-resizer").length
      );
    },
    activeUser() {
      return this.app.activeUser ? this.app.activeUser : null;
    }
  },
  data: () => ({
    isMounted: false,
    pathToUser: null
  }),
  mounted() {
    this.app.preview = this;
    this.isMounted = true;
  },
  methods: {
    getProfileStyle() {
      let userListElt = document.getElementById("userList");
      if (!userListElt) return ``;
      else
        return `
        height: ${userListElt.offsetHeight}px;
      `;
    },
    async getPathToActiveUser(user) {
      return leylo
        .getDocByField("users", "fullName", user.fullName, false)
        .then(realUser => {
          this.pathToUser = `users/${realUser.id}`;
        });
    }
  },
  watch: {
    activeUser(user) {
      // if (user) this.getPathToActiveUser(user);
    },
    // Solely to keep panels are resizer heights uniform
    userList(state) {
      if (!this.isMounted || !this.canRender) return null;
      let userListElt = document.getElementById("userList");
      let userProfileElt = document.getElementById("userProfile");
      if (userProfileElt)
        userProfileElt.style.height = `${userListElt.offsetHeight}px`;
      let panelresizer = document.getElementsByClassName(
        "splitter-pane-resizer"
      );
      panelresizer = panelresizer[0];
      panelresizer.style.height = `${userListElt.offsetHeight}px`;
      panelresizer.style.borderColor = `#fff`;
      panelresizer.style.borderLeft = "5px solid #fff";
      panelresizer.style.opacity = "1";
      panelresizer.style.background = `rgba(0,0,0,0.2)`;
    }
  }
};
</script>

<style>
.profileScreen {
  background-color: #fff;
  height: 100%;
}
.centered {
  display: flex;
  justify-content: center;
  width: 100%;
}
.left-aligned {
  display: flex;
  justify-content: flex-start;
  width: 100%;
}
</style>
