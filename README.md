# v-snackbars
Display the `v-snackbar` (from Vuetify) with a stack display

![Capture](https://user-images.githubusercontent.com/946315/86259071-d8468480-bbbb-11ea-9eca-3aac7d9be455.PNG)

## Requirements

[Vuetify](https://www.npmjs.com/package/vuetify) > v2.3 (it may work with an earlier version of Vuetify but I haven't tested)

## Install

```bash
npm install v-snackbars
```

## Demo

See it in action: https://codesandbox.io/s/v-snackbars-demo-8xrbr?file=/src/App.vue

## How to use

```vue
<v-snackbars :messages.sync="messages"></v-snackbars>
```

You need to provide a `messages` array. Using a `push` on the array will cause the text to be shown in a snackbar.

For example, to display _"This is a message"_, just do the below:
```javascript
this.messages.push("This is a message");
```

You can use the same options as the `v-snackbar`. For example:
```vue
<v-snackbars :messages.sync="messages" timeout="10000" bottom left color="red"></v-snackbars>
```

### Options

#### Snackbar Options

The same [`v-snackbar` options](https://vuetifyjs.com/en/components/snackbars/) should be applicable, like `bottom`, `right`, `left`, `top`, `color`, `timeout`, ….

#### Personalized button

A `close` button is used by default. If you prefer to define your own action button, you can use a ` v-slot:action`.

For example:
```vue
<v-snackbars :messages.sync="messages" timeout="-1" color="black" top right>
  <template v-slot:action="{ close, index, message, id }">
    <v-btn text @click="close(id)">Dismiss</v-btn>
  </template>
</template>
```

By clicking on `Dismiss`, it will remove the related snackbar.

The parameters:
  - `close`: the function to remove a notification
  - `index`: the index in the array of notifications/messages
  - `message`: the current message that is displayed in the notification
  - `id`: the unique key of the message that is used to close it

#### Distance

You can define how much space you want between two snackbars. By default, **55** is used.

For example, if you want more space between each snackbar:
```vue
<v-snackbars :messages.sync="messages" :distance="65"></v-snackbars>
```

## Interactivity

You can add some layers of interactivity with the messages.

For example, you can change the text by doing:
```javascript
this.$set(this.messages, i, "New message to display");
```

To remove a notification, you'll have to change the text to blank `""`:
```javascript
this.$set(this.messages, i, ""); // this item will be removed from "messages" by "v-snackbars"
```

Check the **"Show Interactivity"** button on the [demo](https://codesandbox.io/s/v-snackbars-demo-8xrbr?file=/src/App.vue).
