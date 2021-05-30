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
      :ref="'v-snackbars-'+identifier"
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
      {{topOrBottom[key]}}:{{ calcDistance(key) }}px;
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
      snackbars: [], // array of {key, message, top, right, left, bottom, color, transition, timeout, show}
      keys: [], // array of 'keys'
      heights: {}, // height of each snackbar to correctly position them
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
          timeout:null,
          transition: this.getProp("transition", i) || (right ? 'slide-x-reverse-transition' : 'slide-x-transition'),
          show: false
        });
        this.keys.push(key);
        this.$nextTick(function() {
          this.snackbars[i].show=true; // to see the come-in animation

          this.$nextTick(function () {
            // find the correct height
            let height = this.distance;
            let elem = document.querySelector(".v-snackbars-" + this.identifier + "-" + key);

            if (elem) {
              let wrapper = elem.querySelector(".v-snack__wrapper");
              if (wrapper) {
                height = wrapper.clientHeight + 7;
              }
            }
            this.$set(this.heights, key, height);

            // define the timeout
            let timeout = this.getProp("timeout", i);
            if (timeout > 0) {
              this.snackbars[i].timeout = setTimeout(() => this.removeMessage(key, true), timeout * 1);
            }
          });
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
          // dipose all
          this.keys = this.keys.filter(k => k !== key);
          delete this.heights[key];
          // only send back the changes if it happens from this component
          if (fromComponent) {
            this.$emit(
              "update:messages",
              this.allMessages.filter((m, i) => i !== idx)
            );
            this.$emit("update:objects", this.objects.filter((m, i) => i !== idx));
          }
        }
        // if a timeout on the snackbar, clear it
        if (this.snackbars[idx].timeout) clearTimeout(this.snackbars[idx].timeout);

        // use a timeout to ensure the 'transitionend' will be triggerred
        let timeout = setTimeout(removeSnackbar, 600);
        
        // skip waiting if key does not exist
        let ref = this.$refs['v-snackbars-'+this.identifier];
        if(!ref || !ref[idx]) return;

        // wait the end of the animation
        ref[idx].$el.addEventListener('transitionend', () => {
          clearTimeout(timeout);
          removeSnackbar();
        }, { once: true });
      }
    },
    calcDistance(key) {
      // calculate the position in the stack for the snackbar
      let distance = 0;
      let snackbar = this.snackbars.find((s) => s.key === key);
      if (!snackbar) return 0;
      for (let i = 0; i < this.snackbars.length; i++) {
        // we add all the heights for each visible snackbar in the same corner
        if (this.snackbars[i].key === key) break;
        if (
          this.snackbars[i].show &&
          this.snackbars[i].bottom === snackbar.bottom &&
          this.snackbars[i].top === snackbar.top &&
          this.snackbars[i].right === snackbar.right &&
          this.snackbars[i].left === snackbar.left
        ) {
          distance += this.heights[this.snackbars[i].key] || 0;
        }
      }

      return distance;
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
              if (typeof args[i] === 'string') {
                _this.$set(_this.snackbars[idx], 'message', args[i])
              } else if (typeof args[i] === 'object') {
                for (let prop in args[i]) {
                  if (prop === 'timeout') {
                    let timeout = args[i][prop] * 1;
                    // if there's an existing timeout, clear it before setting the new timeout
                    if (_this.snackbars[idx].timeout) {
                      clearTimeout(_this.snackbars[idx].timeout);
                      _this.snackbars[idx].timeout=null;
                    }
                    if (timeout > -1) {
                      let key = _this.snackbars[idx].key;
                      _this.snackbars[idx].timeout = setTimeout(() => {
                        _this.removeMessage(key, true);
                      }, timeout);
                    }
                  } else {
                    // update the property
                    _this.$set(_this.snackbars[idx], prop, args[i][prop]);
                  }
                }
              }
            }
            idx++;
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
