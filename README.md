# Demo for leylo [![npm version](https://badge.fury.io/js/leylo.svg)](https://badge.fury.io/js/leylo)

| [Installation](https://github.com/Inventsable/leylo#-installation) | [Requirements](https://github.com/Inventsable/leylo#-requirements) | [Usage](https://github.com/Inventsable/leylo#-usage) | [Demo](https://github.com/Inventsable/leylo-test) | [📚 API](https://github.com/Inventsable/leylo#-api) |
| ------------------------------------------------------------------ | :----------------------------------------------------------------: | :--------------------------------------------------: | :-----------------------------------------------: | :-------------------------------------------------: |


Asynchronous utility functions for [Firestore](https://firebase.google.com/docs/firestore/quickstart) within [Vue CLI 3](https://cli.vuejs.org/).

## See [demo site here](https://leylo-test.web.app/)

leylo is imported at the top of the component:

```html
<script>
  import leylo from 'leylo'

  // Component below
  export default ...
</script>
```

Streaming of users is done via simple methods on the component's mounted cycle:

```js
async mounted() {
  this.startAddStream();
  this.startRemoveStream();
},
```

A Boolean is used here to control the circle loading indicator at the top left while intializing the streaming function:

> Notice that if you delete the entire collection with the top right button, the entire stream will fail and detach. Adding users afterward will still present a preview, but our component's list doesn't get updated.

```js
async startAddStream() {
  this.canStream = true;
  this.isStreaming = (await !this.basicStream("users")) ? false : true;
  // this.basicStream() is the same code as addUserStream below

  // If we didn't need this indicator, we could make this more simple, like:
  let addUserStream = await leylo.streamCollection(
    "users",
    user => {
      // If this.userList doesn't already contain this user, add it to our component's data:
      if (!this.userList.some(person => person.fullName == user.fullName))
        this.userList.push(user);
    },
    "added"
  );
},
```

The current flow is done via passing incoming data to a method:

```js
methods: {
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
}
```

There's also a separate stream to handle `removed` documents and filter them from the UI of our component:

```js
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
```
