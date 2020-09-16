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

```js
import VSnackbars from "v-snackbars"
export default {
  components:{
    "v-snackbars": VSnackbars
  },
  […]
}
```

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
<v-snackbars :messages.sync="messages" :timeout="10000" bottom left color="red"></v-snackbars>
```

### Options

#### Snackbar Options

The same [`v-snackbar` options](https://vuetifyjs.com/en/components/snackbars/) should be applicable, like `bottom`, `right`, `left`, `top`, `color`, `timeout`, ….

### Personalized content

You can use `v-slot:default` to customize the content of the snackbar.

For example:
```vue
<v-snackbars :messages.sync="messages" :timeout="-1" color="black" top right>
  <template v-slot:default="{ message }">
    <h3 class="mb-2">Header</h3>
    {{ message }}
  </template>
</v-snackbars>
```

The parameter:
  - `message`: the current message that is displayed in the notification

#### Personalized button

A `close` button is used by default. If you prefer to define your own action button, you can use a ` v-slot:action`.

For example:
```vue
<v-snackbars :messages.sync="messages" :timeout="-1" color="black" top right>
  <template v-slot:action="{ close, index, message, id }">
    <v-btn text @click="close()">Dismiss</v-btn>
  </template>
</v-snackbars>
```

By clicking on `Dismiss`, it will remove the related snackbar.

The parameters:
  - `close`: the function to remove a notification
  - `index`: the index in the array of notifications/messages
  - `message`: the current message that is displayed in the notification
  - `id`: the unique key/id of the message

## Objects

If you want to customize each snackbar, you can also pass a `objects` instead of `messages`, which will contain the various props (like `message`, `color`, `timeout`, `transition` or the position).

In the JavaScript code:
```javascript
this.objects.push({
  message:"Success",
  color:"green",
  timeout:5000
})
this.objects.push({
  message:"Error",
  color:"red",
  timeout:-1
})
```

In your Vue template:
```vue
<v-snackbars :objects.sync="objects"></v-snackbars>
```

Check the **"Random Toast"** button on the [demo](https://codesandbox.io/s/v-snackbars-demo-8xrbr?file=/src/App.vue).

## Interactivity

You can add some layers of interactivity with the messages.

For example, you can change the text by doing:
```javascript
this.$set(this.messages, i, "New message to display");
```

To remove a notification, you'll have to use `splice`:
```javascript
this.messages.splice(i, 1);
```

Check the **"Show Interactivity"** button on the [demo](https://codesandbox.io/s/v-snackbars-demo-8xrbr?file=/src/App.vue).
