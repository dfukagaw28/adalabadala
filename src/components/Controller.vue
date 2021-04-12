<template>
  <div>
    <v-btn
      small
      class="ma-2"
      outlined
      fab
      color="primary"
      @click="onReset"
    >
      <v-icon>mdi-replay</v-icon>
    </v-btn>
    <v-btn
      small
      class="ma-2"
      outlined
      fab
      color="primary"
      @click="play()"
      v-if="isPaused"
    >
      <v-icon>mdi-play</v-icon>
    </v-btn>
    <v-btn
      small
      class="ma-2"
      outlined
      fab
      color="primary"
      @click="pause()"
      v-else
    >
      <v-icon>mdi-pause</v-icon>
    </v-btn>
    <v-btn
      small
      class="ma-2"
      outlined
      fab
      color="primary"
      @click="fit()"
    >
      <v-icon>mdi-arrow-expand</v-icon>
    </v-btn>
  </div>
</template>

<script>
export default {
  props: {
    onReset: {
      type: Function,
      required: true,
    },
  },

  data: () => ({
    isPaused: true,
  }),

  methods: {
    reset() {
      console.log('reset()');
      this.$parent.$refs.canvas.reset();
    },
    play() {
      this.$parent.$refs.canvas.start();
    },
    pause() {
      this.$parent.$refs.canvas.pause();
    },
    fit() {
      console.log('fit');
    },
  },
}
</script>
