<template>
  <v-app
    color="primary"
    dark
  >
    <v-container fluid>
      <v-tabs
        v-model="tab"
      >
        <v-tab>
          Main
        </v-tab>
        <v-tab>
          Settings
        </v-tab>
        <v-tab>
          Output
        </v-tab>
        <v-tab>
          About
        </v-tab>
      </v-tabs>
      <v-tabs-items v-model="tab">
        <v-tab-item>
          <Shuffle
            ref="shuffle"
            :width="width"
            :height="height"
            :items="items"
            :quotas="quotas"
            :ingroup="ingroup"
            :groups="groups"
            :onUpdateGroups="updateGroups"
          />
        </v-tab-item>
        <v-tab-item>
          <Settings
            ref="settings"
            :update-settings="updateSettings"
            :width="width"
            :height="height"
            :items="textItems"
            :quotas="textQuotas"
          />
        </v-tab-item>
        <v-tab-item>
          <v-container>
            <table>
              <thead>
                <tr>
                  <th>Name</th>
                  <th>Group</th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td>item.name</td>
                  <td>ingroup[item.index]</td>
                </tr>
              </tbody>
            </table>
          </v-container>
          {{ ingroup }}
        </v-tab-item>
        <v-tab-item>
          <v-container>
            <p>
              <v-btn
                href="https://github.com/dfukagaw28/adalabadala"
                target="_blank"
                rel="noopener noreferrer"
                text
              >
                <v-icon>mdi-github</v-icon>
                dfukagaw28/adalabadala
              </v-btn>
            </p>
            <p>
              <a
                href="https://www.cis.doshisha.ac.jp/"
                target="_blank"
                rel="noopener noreferrer"
              >
                Faculty of Culture and Information Science,
              </a>
              <br />
              <a
                href="https://www.doshisha.ac.jp/"
                target="_blank"
                rel="noopener noreferrer"
              >
                Doshisha University
              </a>
            </p>
          </v-container>
        </v-tab-item>
      </v-tabs-items>
    </v-container>
  </v-app>
</template>

<script>
import Shuffle from './components/Shuffle';
import Settings from './components/Settings';

const GCOLOR = [
  '#cc0000', '#00cc00', '#0000cc', '#cccc00', '#cc00cc', '#00cccc',
];

export default {
  name: 'App',

  components: {
    Shuffle,
    Settings,
  },

  data: () => ({
    tab: null,
    width: 800,
    height: 600,
    textItems: '桐壺\n帚木\n空蝉\n夕顔\n若紫\n末摘花\n紅葉賀\n花宴\n葵\n賢木\n' +
      '花散里\n須磨\n明石\n澪標\n蓬生\n関屋\n絵合\n松風\n薄雲\n朝顔\n' +
      '少女\n玉鬘\n初音\n胡蝶\n蛍',
    textQuotas: '5\n4\n4\n4\n4\n4',
    items: [],
    quotas: [],
    ingroup: [],
    groups: [],
  }),

  computed: {
  },

  methods: {
    updateSettings(newSettings) {
      for (const key in newSettings) {
        this[key] = newSettings[key];
      }

      const num_items = this.items.length;
      const sum_quotas = this.quotas.reduce((a, b) => a + b);
      let num_groups = this.quotas.length;

      if (num_items == 0) {
        this.ingroup.quotas(0);
        this.ingroup.splice(0);
        return;
      }

      // Update quotas[]
      if (num_items != sum_quotas) {
        let a = Math.min(num_items, (this.quotas.length == 0) ? 5 : this.quotas[0]);
        num_groups = Math.floor(num_items / a);
        a = Math.floor(num_items / num_groups);
        const b = num_items - a * num_groups;
        this.quotas.splice(0);
        for (let j = 0; j < num_groups; j++) {
          this.quotas.push(j < b ? a + 1 : a);
        }
      }

      // Update ingroup[]
      this.ingroup.splice(0);
      for (let j = 0; j < num_groups; j++) {
        for (let k = 0; k < this.quotas[j]; k++) {
          this.ingroup.push(j);
        }
      }

      this.textItems = this.items.join('\n');
      this.textQuotas = this.quotas.join('\n');

      this.$refs.shuffle.resizeCanvas();
    },

    updateGroups(ingroup) {
      const N = this.items.length;
      const M = this.quotas.length;

      // Update ingroup[]
      if (ingroup && (typeof ingroup.length !== 'undefined') && ingroup.length == M) {
        this.ingroup.splice(0, this.ingroup.length, ingroup);
      } else {
        this.ingroup.splice(0);
        let i = 0;
        for (let j = 0; j < M; j++) {
          for (let k = 0; k < this.quotas[j]; k++) {
            this.ingroup[i] = j;
            i++;
          }
        }
        if (i != N) {
          console.log( `WARNING: sum(quotas)=${i} items.length=${N}` );
        }
      }

      // Update groups[]
      this.groups.splice(0);
      for (let j = 0; j < M; j++) {
        const gj = [];
        gj.color = GCOLOR[j % GCOLOR.length];
        this.groups.push(gj);
      }
      for (let i = 0; i < N; i++) {
        this.groups[this.ingroup[i]].push(i);
      }
    },
  },

  mounted() {
    this.items = this.textItems.split('\n');
    this.quotas = this.textQuotas.split('\n').map(s => parseInt(s));
  },
};
</script>
