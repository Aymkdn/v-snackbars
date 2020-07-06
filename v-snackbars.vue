<template>
  <div>
    <v-snackbar
      v-bind="$attrs"
      v-model="snackbar.show"
      :top="snackbar.top"
      :bottom="snackbar.bottom"
      :left="snackbar.left"
      :right="snackbar.right"
      :color="snackbar.color"
      :key="snackbar.key"
      :class="'v-snackbars v-snackbars-'+identifier+'-'+idx"
      :timeout="-1"
      v-for="(snackbar,idx) in snackbars"
    >
      {{ snackbar.message }}
      <template v-slot:action>
        <slot
          name="action"
          :close="removeMessage"
          :id="snackbar.key"
          :index="idx"
          :message="snackbar.message"
        >
          <v-btn text @click="removeMessage(snackbar.key)">close</v-btn>
        </slot>
      </template>
    </v-snackbar>
    <css-style :key="idx" v-for="idx in len">
      .v-snackbars.v-snackbars-{{identifier}}-{{idx}} .v-snack__wrapper {
        transition: {{topOrBottom(idx)}} 500ms;
        {{topOrBottom(idx)}}: 0;
      }
      .v-snackbars.v-snackbars-{{identifier}}-{{idx}} > .v-snack__wrapper {
        {{topOrBottom(idx)}}:{{ idx*distance }}px;
      }
    </css-style>
  </div>
</template>

<script>
export default {
  name: "v-snackbars",
  inheritAttrs: false,
  props: {
    messages: {
      type: Array,
      default: () => []
    },
    timeout: {
      type: [Number, String],
      default: 5000
    },
    distance: {
      type: [Number, String],
      default: 55
    },
    objects: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      len: 0, // we need it to have a css transition
      snackbars: [], // array of {key, message, show(true)}
      identifier: Date.now() + (Math.random() + "").slice(2)
    };
  },
  components: {
    "css-style": {
      render: function(createElement) {
        return createElement("style", this.$slots.default);
      }
    }
  },
  computed: {
    allMessages() {
      if (this.objects.length > 0) return this.objects.map(o => o.message);
      return this.messages;
    }
  },
  watch: {
    messages() {
      this.setSnackbars();
    },
    objects: {
      handler() {
        this.setSnackbars();
      },
      deep: true
    }
  },
  methods: {
    getProp(prop, i) {
      if (typeof this.$attrs[prop] !== "undefined") return this.$attrs[prop];
      if (this.objects.length > i) return this.objects[i][prop];
      if (typeof this[prop] !== "undefined") return this[prop];
      return false;
    },
    topOrBottom(i) {
      return this.getProp("top", i) === true ? "top" : "bottom";
    },
    setSnackbars() {
      // update the text if it changes
      for (let i = 0;i < this.snackbars.length && i < this.allMessages.length; i++) {
        // if the text is blank, then remove the notification
        if (this.allMessages[i] === "") {
          this.removeMessage(this.snackbars[i].key);
          return;
        }
        this.snackbars[i].message = this.allMessages[i];
      }
      for (let i = this.snackbars.length; i < this.allMessages.length; i++) {
        let key = i + "-" + Date.now();
        this.snackbars.push({
          key: key,
          message: this.allMessages[i],
          top: this.getProp("top", i),
          bottom: this.getProp("bottom", i),
          left: this.getProp("left", i),
          right: this.getProp("right", i),
          color: this.getProp("color", i),
          show: true
        });
        let timeout = this.getProp("timeout", i);
        if (timeout > 0) {
          setTimeout(() => this.removeMessage(key), timeout * 1);
        }
      }
      if (this.snackbars.length > this.len) this.len = this.snackbars.length;
    },
    removeMessage(key) {
      let idx = this.snackbars.findIndex(s => s.key === key);
      if (idx > -1) {
        this.snackbars.splice(idx, 1);
        this.$emit("update:messages", this.allMessages.filter((m, i) => i !== idx));
        this.$emit("update:objects", this.objects.filter((m, i) => i !== idx));
      }
    }
  },
  created() {
    this.setSnackbars();
  }
};
</script>
