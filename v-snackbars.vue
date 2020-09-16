<template>
  <div>
    <v-snackbar
      v-bind="$attrs"
      v-model="snackbar.show"
      :transition="snackbar.transition"
      :top="snackbar.top"
      :bottom="snackbar.bottom"
      :left="snackbar.left"
      :right="snackbar.right"
      :color="snackbar.color"
      :key="snackbar.key"
      :ref="'snackbar-'+snackbar.key"
      :class="'v-snackbars v-snackbars-'+identifier+'-'+snackbar.key"
      :timeout="-1"
      v-for="(snackbar,idx) in snackbars"
    >
      <template v-slot:default>
        <slot :message="snackbar.message">
          {{ snackbar.message }}
        </slot>
      </template>
      <template v-slot:action>
        <slot
          name="action"
          :close="() => removeMessage(snackbar.key, true)"
          :id="snackbar.key"
          :index="idx"
          :message="snackbar.message"
        >
          <v-btn text @click="removeMessage(snackbar.key, true)">close</v-btn>
        </slot>
      </template>
    </v-snackbar>
    <css-style :key="key+idx" v-for="(key,idx) in keys">
      .v-snackbars.v-snackbars-{{identifier}}-{{key}} .v-snack__wrapper {
      transition: {{topOrBottom[key]}} 500ms;
      {{topOrBottom[key]}}: 0;
      }
      .v-snackbars.v-snackbars-{{identifier}}-{{key}} > .v-snack__wrapper {
      {{topOrBottom[key]}}:{{ indexPosition[key]*distances[key] }}px;
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
      keys: [], // array of 'keys'
      distances: {}, // height of each snackbar to correctly position them
      identifier: Date.now() + (Math.random() + "").slice(2) // to avoid issues when several v-snackbars on the page
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
    },
    // to correcly position the snackbar
    indexPosition() {
      let ret = {};
      let idx = {
        topCenter: 0,
        topLeft: 0,
        topRight: 0,
        bottomCenter: 0,
        bottomLeft: 0,
        bottomRight: 0
      };
      this.snackbars.forEach(o => {
        if (o.top && !o.left && !o.right) ret[o.key]=idx.topCenter++;
        if (o.top && o.left) ret[o.key]=idx.topLeft++;
        if (o.top && o.right) ret[o.key]=idx.topRight++;
        if (o.bottom && !o.left && !o.right) ret[o.key]=idx.bottomCenter++;
        if (o.bottom && o.left) ret[o.key]=idx.bottomLeft++;
        if (o.bottom && o.right) ret[o.key]=idx.bottomRight++;
      });
      return ret;
    },
    topOrBottom() {
      let ret = {};
      this.snackbars.forEach(o => {
        ret[o.key] = (o.top ? 'top' : 'bottom');
      })
      return ret;
    }
  },
  watch: {
    messages() {
      this.eventify(this.messages);
    },
    objects: {
      handler() {
        this.eventify(this.objects);
      },
      deep: true
    },
    snackbars() {
      // retrieve the height for each snackbar
      this.$nextTick(function() {
        let ret = {};
        let len = this.snackbars.length;
        this.snackbars.forEach((o, idx) => {
          let distance = this.distance;
          if (idx + 1 < len) {
            let nextKey = this.snackbars[idx + 1].key;
            let elem = document.querySelector(".v-snackbars-" + this.identifier + "-" + o.key);
            if (elem) {
              let wrapper = elem.querySelector(".v-snack__wrapper");
              if (wrapper) {
                distance = wrapper.clientHeight + 7;
              }
            }
            ret[nextKey] = distance;
          }
        });
        this.distances = ret;
      });
    }
  },
  methods: {
    getProp(prop, i) {
      if (this.objects.length > i && typeof this.objects[i][prop] !== 'undefined')
        return this.objects[i][prop];
      if (typeof this.$attrs[prop] !== 'undefined') return this.$attrs[prop];
      if (typeof this[prop] !== 'undefined') return this[prop];
      return undefined;
    },
    setSnackbars() {
      for (let i = this.snackbars.length; i < this.allMessages.length; i++) {
        let key = i + "-" + Date.now();
        let top = this.getProp("top", i);
        let bottom = this.getProp("bottom", i);
        let left = this.getProp("left", i);
        let right = this.getProp("right", i);
        top = (top===""?true:top);
        bottom = (bottom===""?true:bottom);
        left = (left===""?true:left);
        right = (right===""?true:right);
        // by default, it will be at the bottom
        if (!bottom && !top) bottom=true;
        this.snackbars.push({
          key: key,
          message: this.allMessages[i],
          top: top,
          bottom: bottom,
          left: left,
          right: right,
          color: this.getProp("color", i) || "black",
          transition: this.getProp("transition", i) || (right ? 'slide-x-reverse-transition' : 'slide-x-transition'),
          show: false
        });
        this.keys.push(key);
        this.$nextTick(function() {
          this.snackbars[i].show=true; // to see the come-in animation
          let timeout = this.getProp("timeout", i);
          if (timeout > 0) {
            setTimeout(() => this.removeMessage(key, true), timeout * 1);
          }
        })
      }
    },
    removeMessage(key, fromComponent) {
      let idx = this.snackbars.findIndex(s => s.key === key);
      if (idx > -1) {
        this.snackbars[idx].show=false;

        let removeSnackbar = () => {
          let idx = this.snackbars.findIndex(s => s.key === key);
          this.snackbars.splice(idx, 1);
          // only send back the changes if it happens from this component
          if (fromComponent) {
            this.$emit(
              "update:messages",
              this.allMessages.filter((m, i) => i !== idx)
            );
            this.$emit("update:objects", this.objects.filter((m, i) => i !== idx));
          }
        }
        // use a timeout to ensure the 'transitionend' will be triggerred
        let timeout = setTimeout(removeSnackbar, 600);
        // wait the end of the animation
        this.$refs['snackbar-'+key][0].$el.addEventListener('transitionend', () => {
          clearTimeout(timeout);
          removeSnackbar();
        }, { once: true });
      }
    },
    eventify (arr) {
      // detect changes on 'messages' and 'objects'
      let _this = this;
      let eventify = function(arr) {
        arr.isEventified = true;
        // overwrite 'push' method
        let pushMethod = arr.push;
        arr.push = function(e) {
          pushMethod.call(arr, e);
          _this.setSnackbars();
        };
        // overwrite 'splice' method
        let spliceMethod = arr.splice;
        arr.splice = function() {
          let args = [], len = arguments.length;
          while (len--) args[len] = arguments[len];
          spliceMethod.apply(arr, args);
          let idx = args[0];
          let nbDel = args[1];
          let elemsLen = args.length - 2;

          // do we just remove an element?
          if (elemsLen === 0) {
            nbDel += idx;
            while(idx < nbDel) {
              if (_this.snackbars[idx]) {
                _this.removeMessage(_this.snackbars[idx].key);
              }
              idx++;
            }
          }
          else if (elemsLen > 0) {
            // or we set a value on an element using this.$set, so we update the message
            for (let i=2; i<elemsLen+2; i++) {
              if (_this.snackbars[idx]) _this.$set(_this.snackbars[idx], 'message', args[i]);
              idx++;
            }
          }
        };
      };
      if (!arr.isEventified) eventify(arr);
    }
  },
  created() {
    this.eventify(this.messages);
    this.eventify(this.objects);
  }
};
</script>
