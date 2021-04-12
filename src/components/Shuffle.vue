<template>
  <v-container fluid>
    <v-row class="text-center">
      <v-col md="8" sm="12">
        <v-container>
          <v-row>
            <v-col>
              <Controller
                ref="controller"
                :onReset="reset"
              />
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <Canvas
                ref="canvas"
                :width="width"
                :height="height"
                :balls="itemObjs"
                :quotas="quotas"
                :groups="groups"
                :ingroup="ingroup"
                :onUpdateGroups="onUpdateGroups"
              />
            </v-col>
          </v-row>
        </v-container>
      </v-col>
      <v-col md="4" sm="12">
        <Table
          :items="itemObjs"
          :groups="groups"
          :ingroup="ingroup"
        />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import Canvas from './Canvas.vue';
import Controller from './Controller.vue';
import Table from './Table.vue';

const COLORS = [
  '#ff2800', '#faf500', '#35a16b', '#0041ff', '#66ccff',
  '#ff99a0', '#ff9900', '#9a0079', '#663300',
  '#ffd1d1', '#ffff99', '#cbf266', '#b4ebfa', '#edc58f',
  '#87e7b0', '#c7b2de', '#c8c8cb', '#7f878f',
];

export default {
  name: 'Shuffle',

  components: {
    Canvas,
    Controller,
    Table,
  },

  props: {
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
    items: {
      type: Array,
      required: true,
    },
    quotas: {
      type: Array,
      required: true,
    },
    ingroup: {
      type: Array,
      required: true,
    },
    groups: {
      type: Array,
      required: true,
    },
    onUpdateGroups: {
      type: Function,
      required: true,
    }
  },

  data: () => ({
    isPaused: true,
    itemObjs: [],
  }),

  methods: {
    reset() {
      console.log( 'shuffle reset()' );
      console.log( this.items );

      const N = this.items.length;

      // Initialize items
      this.itemObjs.splice(0);
      for (let i = 0; i < N; i++) {
        this.itemObjs.push({
          index: i,
          name: this.items[i],
          color: COLORS[i % COLORS.length],
        });
      }
      console.log( this.itemObjs );

      this.onUpdateGroups();

      if (typeof this.$refs.canvas !== 'undefined') {
        console.log('xxx')
        this.$refs.canvas.reset();
      }
    },

    play() {
      this.$refs.canvas.start();
      this.isPaused = false;
    },

    pause() {
      this.$refs.canvas.pause();
      this.isPaused = true;
    },

    fit() {
      console.log('fit');
    },

    resizeCanvas() {
      this.$refs.canvas.resize();
    },
  },

  created() {
    console.log( 'shuffle created()' );
    this.reset();
  },
}
</script>
