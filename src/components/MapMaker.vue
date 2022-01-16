<template>
  <div class="main-wrapper">
    <div class="map">
      <simple-map :settings="settings" :items="countries" :geography="$options.countriesData"/>
    </div>
    <div class="controls">
      <div class="color-control">
        <label>Background color:</label>
        <input name="Color Picker"
        type="color" v-model="backgroundColor"/>
        <input v-model.lazy="backgroundColor" @input="isColor($event)">
      </div>
      <div class="color-control">
        <label>Primary fill color:</label>
        <input name="Color Picker"
        type="color" v-model="primaryFill"/>
        <input v-model.lazy="primaryFill" @input="isColor($event)">
      </div>
      <div class="color-control">
        <label>Primary stroke color:</label>
        <input name="Color Picker"
        type="color" v-model="primaryStrokeColor"/>
        <input v-model.lazy="primaryStrokeColor" @input="isColor($event)">
      </div>
      <div class="color-control">
        <label>Focus color:</label>
        <input name="Color Picker"
        type="color" v-model="focusColor"/>
        <input v-model.lazy="focusColor" @input="isColor($event)">
      </div>
    </div>
    <div class="data-explorer">
      numeric fields: {{quantitativeFields}}
      qualitative fields: {{qualitativeFields}}
    </div>
  </div>
</template>

<script>
import { ckmeans } from 'simple-statistics';
import COUNTRIES from '@/map/country_index.json';
import COUNTRIES_LOOKUP_JSON from '@/map/country_paths_lookup.json';
import SimpleMap from './SimpleMap.vue';

export default {
  components: { SimpleMap },
  name: 'MapMaker',
  countriesData: COUNTRIES_LOOKUP_JSON,
  props: {
    msg: String,
  },
  data() {
    return {
      backgroundColor: '#b3f0ff',
      highlightColor: '#ffffff',
      primaryFill: '#27251b',
      primaryStrokeColor: '#111111',
      focusColor: '#ec7c13',
      countries: [],
    };
  },
  computed: {
    fields() {
      console.log('computing fields');
      const numRows = this.countries.length;
      console.log(numRows);
      const fields = [];
      const excludeFields = ['name', 'id'];
      // get fields to check
      if (numRows > 0) {
        for (const property in this.countries[0]) { // eslint-disable-line
          console.log(property);
          if (!excludeFields.includes(property)) {
            fields.push(property);
          }
        }
      }
      return fields;
    },
    quantitativeFields() {
      return this.fields.filter((field) => this.isQuantitative(field, this.countries));
    },
    qualitativeFields() {
      return this.fields.filter((field) => !this.quantitativeFields.includes(field));
    },
    settings() {
      return {
        backgroundColor: this.backgroundColor,
        highlightColor: this.highlightColor,
        primaryFill: this.primaryFill,
        primaryStrokeColor: this.primaryStrokeColor,
        focusColor: this.focusColor,
      };
    },
  },
  watch: {
    backgroundColor(newval, oldval) {
      const rx = /^#([0-9a-f]{3}|[0-9a-f]{6})$/i;
      if (!rx.test(newval)) {
        this.backgroundColor = oldval;
      }
    },
  },
  methods: {
    isQuantitative(property, rows) {
      console.log('in isnumeric');
      console.log(property);
      if (rows.every((row) => Number(row[property]))) {
        console.log('returning true');
        return true;
      }
      console.log('returning false');
      return false;
    },
    generateBreaks() {
      ckmeans();
    },
    isColor(evt) {
      console.log(evt);
      console.log('is it a color');
    },
    isNumber(evt) {
      console.log(evt);
      const charCode = evt.which ? evt.which : evt.keyCode;
      if (
        charCode > 31
        && (charCode < 48 || charCode > 57)
        && charCode !== 46
      ) {
        evt.preventDefault();
      }
    },
    handleClick(countryId) {
      console.log(countryId);
    },
    handleHover(countryId) {
      this.$refs[countryId][0].setAttribute('fill', this.highlightColor);
    },
  },
  mounted() {
    this.countries.push(...COUNTRIES);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main-wrapper {
  display: grid;
  grid-template-columns: 1fr auto;
}

@media only screen and (max-width: 1100px) {
  .main-wrapper {
    grid-template-columns: 1fr;
  }
}

.data-explorer {
  grid-column: 1 / 3;
}

.country {
    fill: #cccccc;
}

.color-control {
  display: grid;
  grid-template-columns: repeat(3, auto);
  align-items: center;
  justify-content: center;
  gap: 10px;
}
</style>
