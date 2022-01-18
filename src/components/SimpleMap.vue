<template>
  <div class="map-container">
    <div class="legend" :style="legendStyle()">
      <ul>
      <li class="legend-item" v-for="(bin, index) in settings.bins" :key="index">
        <div :style="patchStyle(bin.color, symbolWidth, symbolHeight)"></div>
        <div class="legend-label">{{ bin.label }}</div>
      </li>
      </ul>
    </div>
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
          v-for="item in geography" :key="item.id"
          @click="handleClick(item.id)"
          @mouseenter="handleHover(item.id)"
          @mouseleave="handleLeave(item.id)"
          :style="countryStyle(item.id)"
          view
          :id=item.id
          :ref=item.id
          :d="item.d"
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
    displayField: Object,
  },
  data() {
    return {
      initializedMap: false,
      symbolHeight: 15,
      symbolWidth: 10,
      focusedItem: null,
      displaySettings: {},
      displayItems: [],
    };
  },
  computed: {
    noDataStyle() {
      return {
        fill: this.settings.colors.noData.color,
        transition: '.2s fill',
        stroke: this.primaryStrokeColor,
        'stroke-width': this.primaryStrokeWidth,
      };
    },
    backgroundColor() {
      return this.settings.colors.background.color;
    },
    focusColor() {
      return this.settings.colors.focus.color;
    },
    primaryStrokeColor() {
      return this.settings.colors.primaryStroke.color;
    },
    primaryFill() {
      return this.settings.colors.noData.color;
    },
    primaryStrokeWidth() {
      return this.settings.primaryStrokeWidth;
    },
  },
  watch: {
    items() {
      this.initMap();
    },
    settings() {
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
    patchStyle(color) {
      return {
        'background-color': color,
        width: `${this.settings.legend.patch.width}px`,
        height: `${this.settings.legend.patch.height}px`,
      };
    },
    getFill(item) {
      if (this.displayField.type === 'quantitative') {
        let color = null;
        this.settings.bins.some((bin) => {
          if (item[this.displayField.name] >= bin.min
          && item[this.displayField.name] <= bin.max) {
            color = bin.color;
            return true;
          } return false;
        });
        return color;
      }
      if (this.displayField.type === 'qualitative') {
        let color = null;
        this.settings.bins.some((bin) => {
          if (bin.values.includes(item[this.displayField.name])) {
            color = bin.color;
            return true;
          }
          return false;
        });
        return color;
      }
      // should be unreachable
      return this.settings.colors.noData.color;
    },
    legendStyle() {
      const fColor = this.settings.legend.colors.background.color;
      return {
        position: 'absolute',
        bottom: 0,
        margin: '1em',
        padding: '0.5em',
        'background-color': `rgb(${fColor.r}, ${fColor.g}, ${fColor.b}, ${fColor.a})`,
        'border-radius': '3px',
      };
    },
    countryStyle(countryId) {
      let fillColor = '';
      if (!(countryId in this.displaySettings)) {
        return this.noDataStyle;
      }
      if (countryId === this.focusedItem || this.displaySettings[countryId].focused) {
        fillColor = this.settings.colors.focus.color;
      } else {
        fillColor = this.displaySettings[countryId].fill;
      }
      return {
        fill: fillColor,
        transition: '.2s fill',
        stroke: this.primaryStrokeColor,
        'stroke-width': this.primaryStrokeWidth,
      };
    },
    handleClick(countryId) {
      if (!this.displayItems.length) { return false; }
      this.focusedItem = countryId;
      return true;
    },
    handleHover(countryId) {
      if (!this.displayItems.length) { return false; }
      if (!this.displaySettings[countryId]) {
        console.log(countryId);
        return false;
      }
      this.displaySettings[countryId].focused = true;
      return true;
    },
    handleLeave(countryId) {
      if (!this.displayItems.length) { return false; }
      this.displaySettings[countryId].focused = false;
      return true;
    },
    initMap() {
      const displayableItems = this.items.filter((row) => row.id in this.geography);
      displayableItems.forEach((element) => {
        const style = { focused: false, highlighted: false, fill: this.getFill(element) };
        this.$set(this.displaySettings, element.id, style);
      });
      this.displayItems = displayableItems;
    },
  },
  mounted() {
    this.initMap();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul {
  list-style: none;
  padding: 0px;
}

.map-container {
  position: relative;
}

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

.legend-item {
  display: flex;
}

.legend-label {
  padding-left: 5px;
}

</style>
