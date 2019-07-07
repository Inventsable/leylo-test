<template>
  <v-container>
    <v-layout row>
      <v-flex xs12 sm10 offset-sm1 md8 offset-md2>
        <v-card>
          <v-toolbar flat dark style="z-index: 1000;">
            <v-btn icon :loading="isStreaming" @click="ensureStreamStarts(true)">
              <v-icon>mdi-account-convert</v-icon>
            </v-btn>
            <v-toolbar-title>{{`${isStreaming ? 'Streaming' : 'No longer streaming'} users/`}}</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon @click="generateRandomProfile()">
              <v-icon>mdi-account-plus</v-icon>
            </v-btn>
            <v-btn icon :disabled="!userList.length" @click="deleteAllUsers()">
              <v-icon>mdi-account-multiple-minus</v-icon>
            </v-btn>
          </v-toolbar>
          <div>
            <split-pane
              v-if="!mobileDevice"
              :min-percent="20"
              :default-percent="50"
              split="vertical"
            >
              <template slot="paneL">
                <userlist :list="userList" />
              </template>
              <template slot="paneR">
                <userprofile />
              </template>
            </split-pane>
            <div v-else>
              <userlist :list="userList" />
            </div>
          </div>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
// STILL USING CLIENT LEYLO
import leylo from "leylo";
import { isMobile, isLandscape } from "@inventsable/simpledevice";

// Various UI components/helpers
import userlist from "./userlist";
import userprofile from "./userprofile";
import splitPane from "vue-splitpane";

// Various Node helpers
// import ThisPersonDoesNotExist from "thispersondoesnotexist-js";
// const dnte = new ThisPersonDoesNotExist();
const randomAvatar = require("random-avatar");
const randomProfile = require("random-profile-generator");

export default {
  components: {
    userlist,
    userprofile,
    "split-pane": splitPane
  },
  data: () => ({
    local: false,
    userList: [],
    isStreaming: false,
    isAltStreaming: false,
    canStream: true,
    mobileDevice: null,
    userStream: null
  }),
  computed: {
    app() {
      return this.$root.$children[0];
    }
  },
  watch: {
    userList(entries) {
      if (entries.length == 0) this.isStreaming = false;
    }
  },
  async mounted() {
    this.app.main = this;

    // In case collection 'users' is empty on launch
    this.ensureStreamStarts().then(() => {
      this.startAddStream();
      this.startRemoveStream();
    });

    this.mobileDevice = this.checkIfMobile();
    window.addEventListener("resize", () => {
      this.mobileDevice = this.checkIfMobile();
    });
  },
  methods: {
    async ensureStreamStarts(forceStream = false) {
      let users = await leylo.getCollection("users");
      if (!users) {
        let newUser = this.generateRandomProfile();
        if (forceStream)
          newUser.then(() => {
            this.startAddStream();
            this.startRemoveStream();
          });
      } else {
        this.startAddStream();
        this.startRemoveStream();
      }
    },
    async startAddStream() {
      this.canStream = true;
      this.isStreaming = (await !this.basicStream("users")) ? false : true;
    },
    async startRemoveStream() {
      return await leylo.streamCollection(
        "users",
        user => {
          this.userList = this.userList.filter(item => {
            return item.fullName !== user.data().fullName;
          });
        },
        "removed",
        false
      );
    },
    async basicStream(collection) {
      return this.canStream
        ? await leylo.streamCollection(
            collection,
            this.addUserIfNotInList,
            "added"
          )
        : null;
    },
    addUserIfNotInList(user) {
      if (!this.userList.some(person => person.fullName == user.fullName)) {
        this.userList.push(user);
      }
    },
    checkIfMobile() {
      return (
        window.innerWidth < 800 ||
        typeof window.orientation !== "undefined" ||
        navigator.userAgent.indexOf("IEMobile") !== -1
      );
    },
    async startAltStream() {
      this.userStream = await leylo.streamPath(
        "users",
        user => {
          console.log(`Hello ${user.data().firstName}`);
          return `Hello ${user.data().firstName}`;
        },
        "added",
        false
      );
      this.isAltStreaming = true;
    },
    async stopAltStream() {
      let unsubscribe = this.userStream();
      this.isAltStreaming = false;
    },
    stopStream() {
      this.canStream = false;
    },
    async generateRandomProfile() {
      let newUser = randomProfile.profile();
      return await leylo.addDoc("users", newUser).then(ref => {
        leylo.getDocById("users", ref).then(doc => {
          this.app.activeUser = doc;
        });
      });
    },
    async deleteAllUsers() {
      this.app.activeUser = null;
      this.isStreaming = false;
      this.canStream = true;

      let fulldeletion = await leylo.deleteCollection("users");
      let completed = false;
      fulldeletion.forEach(result => {
        if (!result) completed = false;
      });
      if (completed) return (this.userList = []);
    },
    async demonstrateFieldDeletion() {
      let deletion = await leylo.deleteFieldByDocId(
        "users",
        "Inventsable",
        "location"
      );
      console.log(deletion);
    }
  },
  created() {
    // if (!this.local) this.validateDBEntry(this.$route.params.id);
  }
};
</script>

<style>
.demotoolbar {
  display: flex;
  justify-content: space-around;
  padding: 10px 10%;
  background-color: rgba(0, 0, 0, 0.05);
}

.sample {
  background-color: red;
  width: 100%;
  height: 100%;
}

.vue-splitter-container[data-v-566a42b8] {
  position: inherit;
}
</style>
