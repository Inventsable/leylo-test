<template>
  <v-list id="userList" two-line>
    <template v-for="item in list">
      <v-list-tile :key="item.fullName" avatar @click="selectUser(item)">
        <v-list-tile-avatar>
          <img :src="item.avatar" />
        </v-list-tile-avatar>
        <v-list-tile-content>
          <v-list-tile-title v-html="item.fullName"></v-list-tile-title>
          <v-list-tile-sub-title v-html="item.phone"></v-list-tile-sub-title>
        </v-list-tile-content>
        <v-list-tile-action>
          <v-btn icon @click="deleteThisUser(item)">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-list-tile-action>
      </v-list-tile>
    </template>
  </v-list>
</template>

<script>
import leylo from "leylo";

export default {
  name: "userlist",
  props: {
    list: Array
  },
  computed: {
    app() {
      return this.$root.$children[0];
    },
    activeUser() {
      return this.app.activeUser;
    }
  },
  methods: {
    selectUser(user) {
      // userprofile.vue is watching this and changes automatically
      this.app.activeUser = user;
    },
    deleteThisUser(user) {
      // Counts as one read -- find the user's document by some field value
      return leylo
        .getDocByField("users", "fullName", user.fullName, false)
        .then(realUser => {
          if (this.activeUser.fullName == user.fullName)
            this.app.activeUser = null;
          // Then use this data to find exact location for deletion
          leylo.deleteDocById("users", realUser.id);
        });
    },
    async deleteAsync(user) {}
  }
};
</script>

<style>
#userList {
  min-height: 400px;
}
</style>
