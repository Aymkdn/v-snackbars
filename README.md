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

It will be very similar to the `v-snackbar` (you can use the same options):
```vue
<v-snackbars :messages.sync="messages" timeout="6000" color="blue" bottom right></v-snackbars>
```

You need to provide a `messages` array. Using a `push` on the array will cause the text to be shown in a snackbar.

For example, to display "This is a message", just do the below:
```javascript
this.messages.push("This is a message");
```

### Options

#### Personalized button

A `close` button is used by default. If you prefer to define your own action button, you can use a ` v-slot:action`.
For example:
```vue
<v-snackbars :messages.sync="messages" timeout="-1" color="black" top right>
  <template v-slot:action="{ close, index }">
    <v-btn text @click="close(index)">Dismiss</v-btn>
  </template>
</template>
```

By clicking on `Dismiss`, it will remove the related snackbar.

#### Distance

You can define how much space you want between two snackbars. By default, **55** is used.

For example, if you want more space between each snackbar:
```vue
<v-snackbars :messages.sync="messages" timeout="-1" top left :distance="65"></v-snackbars>
```
