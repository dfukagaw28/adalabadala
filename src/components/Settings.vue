<template>
  <v-container>
    <v-row>
      <v-col>
        <v-text-field
          label="Width"
          :value="width"
          @input="updateWidth"
        ></v-text-field>
      </v-col>
      <v-col>
        <v-text-field
          label="Height"
          :value="height"
          @input="updateHeight"
        ></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-textarea
          label="Items"
          auto-grow
          :value="items"
          @input="updateItems"
        ></v-textarea>
      </v-col>
      <v-col>
        <v-textarea
          label="Group sizes"
          auto-grow
          :value="quotas"
          @input="updateQuotas"
        ></v-textarea>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
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
      type: String,
      required: true,
    },
    quotas: {
      type: String,
      required: true,
    },
    updateSettings: {
      type: Function,
      required: true,
    },
  },
  methods: {
    updateWidth(newValue) {
      let val = parseInt(newValue, 10);
      if (isNaN(val)) { val = 800; }
      this.updateSettings({ width: val });
    },
    updateHeight(newValue) {
      let val = parseInt(newValue, 10);
      if (isNaN(val)) { val = 600; }
      this.updateSettings({ height: val });
    },
    updateItems(newValue) {
      let val = this.text2array(newValue);
      this.updateSettings({ items: val });
    },
    updateQuotas(newValue) {
      let val = this.text2array(newValue);
      val = val.map(s => parseInt(s));
      val = val.filter(x => !isNaN(x));
      val.sort((a, b) => b - a);
      this.updateSettings({ quotas: val });
    },
    text2array(s) {
      return s.trim().replace(/\s+/gm, ' ').split(' ');
    },
  },
}
</script>
