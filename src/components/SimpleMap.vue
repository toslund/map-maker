<template>
  <div class="">
    <svg
      width="100%" height="100%"
      x="190" y="129"
      viewBox="190 129 1060 471"
      preserveAspectRatio="xMinYMin"
    >
      <rect preserveAspectRatio="xMinYMin"
      x="190" y="129" :fill="backgroundColor"
     width="100%" height="100%"></rect>
      <circle fill="#555555" cx="0" cy="0" r="50"/>
      <!-- <circle fill="#555555" cx="190" cy="129" r="50"/> -->
      <g
        class="map"
      >
        <path
          v-for="country in displayItems" :key="country.id"
          @click="handleClick(country.id)"
          @mouseenter="handleHover(country.id)"
          @mouseleave="handleLeave(country.id)"
          :style="countryStyle(country.id)"
          view
          :id=country.id
          :ref=country.id
          :d="geography[country.id].d"
          :fill="primaryFill"
        ></path>
      </g>
    </svg>
  </div>
</template>

<script>

export default {
  name: 'SimpleMap',
  props: {
    settings: Object,
    items: Array,
    geography: Object,
    categories: Array,
    datatype: String,
    datakey: String,
  },
  data() {
    return {
      focusedItem: null,
      displaySettings: {},
      displayItems: [],
    };
  },
  computed: {
    highlightedCountryStyle() {
      return {
        fill: this.highlightColor,
      };
    },
    focusedCountryStyle() {
      return {
        fill: this.focusColor,
        transition: '.2s fill',
      };
    },
    baseCountryStyle() {
      return {
        fill: this.primaryFill,
        transition: '.2s fill',
        stroke: this.primaryStrokeColor,
      };
    },
    backgroundColor() {
      return this.settings.backgroundColor;
    },
    highlightColor() {
      return this.settings.highlightColor;
    },
    focusColor() {
      return this.settings.focusColor;
    },
    primaryStrokeColor() {
      return this.settings.primaryStrokeColor;
    },
    primaryFill() {
      return this.settings.primaryFill;
    },
  },
  watch: {
    items() {
      this.initMap();
    },
    backgroundColor(newval, oldval) {
      const rx = /^#([0-9a-f]{3}|[0-9a-f]{6})$/i;
      if (!rx.test(newval)) {
        this.backgroundColor = oldval;
      }
    },
  },
  methods: {
    countryStyle(countryId) {
      if (countryId === this.focusedItem || this.displaySettings[countryId].focused) {
        return this.focusedCountryStyle;
      }
      if (this.displaySettings[countryId].highlighted) {
        return this.highlightedCountryStyle;
      }
      return this.baseCountryStyle;
    },
    handleClick(countryId) {
      console.log(countryId);
      this.focusedItem = countryId;
    },
    handleHover(countryId) {
      this.displaySettings[countryId].focused = true;
    },
    handleLeave(countryId) {
      this.displaySettings[countryId].focused = false;
    },
    initMap() {
      this.items.forEach((element) => {
        const style = { focused: false, highlighted: false };
        this.$set(this.displaySettings, element.id, style);
      });
      this.displayItems = this.items;
    },
  },
  mounted() {
    this.initMap();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.country {
    fill: #cccccc;
}

.color-control {
  display: grid;
  grid-template-columns: repeat(3, min-content);
  align-items: center;
  justify-content: center;
  gap: 10px;
}
</style>
