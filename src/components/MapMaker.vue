<template>
  <div class="main-wrapper">
    <div class="map">
      <simple-map
      ref="map"
      :settings="settings"
      :items="items"
      :geography="$options.countriesData"
      :displayField="displayField"/>
    </div>
    <div class="controls">
      <div class="tabs-menu">
        <div class="tabs">
          <button
            class="tab"
            :class="{ 'active-tab': symbologyTab === 'general-color' }"
            @click="symbologyTab='general-color'">
            Style
          </button>
          <button
          class="tab"
          :class="{ 'active-tab': symbologyTab === 'classification-color' }"
          @click="symbologyTab='classification-color'">
            Symbols
          </button>
          <button
          class="tab"
          :class="{ 'active-tab': symbologyTab === 'legend-settings' }"
          @click="symbologyTab='legend-settings'">
            Legend
          </button>
        </div>
      </div>
      <div id="color-controls" v-show="symbologyTab === 'general-color'">
        <div class="color-controls">
          <div class="color-control" v-for="(color, index) in colors"
          :key="index"
          @click="activeColor=color.key">
            <span
            :style="patchStyle(color.color)">
            </span>
            <label
            :class="{'color-control-label': true, 'active': activeColor===color.key}">
              {{color.name}}
            </label>
          </div>
        </div>
        <div class="color-picker">
        <v-color-picker v-model="colors[activeColor].color"></v-color-picker>
        </div>
      </div>
      <div id="bin-controls" v-show="symbologyTab === 'classification-color'" v-if="bins.length">
        <div class="color-controls">
          <div class="color-control" v-for="(bin, index) in bins"
          :key="index"
          @click="activeBin=index">
            <span
            :style="patchStyle(bin.color)">
            </span>
            <label
              :class="{'color-control-label': true, 'active': activeBin===index}">
            {{ bin.label }}
            </label>
          </div>
        </div>
        <div class="color-picker">
        <v-color-picker v-model="bins[activeBin].color"></v-color-picker>
        </div>
      </div>
      <div id="bin-controls" v-show="symbologyTab === 'legend-settings'">
        <div class="color-controls">
          <div class="color-control" v-for="(color, index) in legend.colors"
          :key="index"
          @click="activeLegendColor=color.key">
            <span
            :style="patchStyle(color.color)">
            </span>
            <label
            :class="{'color-control-label': true, 'active': activeLegendColor===color.key}">
              {{color.name}}
            </label>
          </div>
        </div>
        <div class="color-picker">
        <v-color-picker mode="rgba" v-model="legend.colors[activeLegendColor].color"></v-color-picker>
        </div>
        <div class="color-controls">
        <div class="color-control">
          <label>Patch Width:</label>
          <v-text-field
              dense
              hide-details="auto"
              outlined
              type="number"
              step="1"
              min="5"
              max="50"
              ref="patchWidth"
              v-model.number="legend.patch.width"
            ></v-text-field>
        </div>
        <div class="color-control">
          <label>Patch Height:</label>
          <v-text-field
              dense
              hide-details="auto"
              outlined
              type="number"
              step="1"
              min="5"
              max="50"
              ref="patchWidth"
              v-model.number="legend.patch.height"
            ></v-text-field>
        </div>
        </div>
      </div>
    </div>
    <div class="data-explorer">
      <div class="tabs-menu">
        <div class="tabs">
          <button
            class="tab"
            :class="{ 'active-tab': dataExplorerTab === 'data-source' }"
            @click="dataExplorerTab='data-source'">
            Data Source
          </button>
          <button
          class="tab"
          :class="{ 'active-tab': dataExplorerTab === 'classification' }"
          @click="dataExplorerTab='classification'">
            Classification
          </button>
        </div>
      </div>
      <div v-show="dataExplorerTab==='data-source'">
        <div class="data-source-tab">
          <div class="file-input">
            <v-file-input
            v-model="file"
              accept=".csv"
              label="File input"
            ></v-file-input>
            <v-btn depressed @click="loadExampleCSV">
              Reset Example Data
            </v-btn>
            <v-btn depressed @click="downloadExample">
              Download Example CSV
            </v-btn>
          </div>
        </div>
      </div>
      <div v-show="dataExplorerTab==='classification'">
        <div class="classification-tab">
          <div>
          <v-select
          :items="fields"
          item-text="text"
          return-object
          label="Display Field"
          v-model="displayField"
        ></v-select>
        <v-select
          :full-width="false"
          :items="allColorSchemes"
          v-model="activeColorScheme"
          @change="refreshSettings"
          item-text="text"
          return-object
          label="Color Palette"
          hint="Customize color in the symbology tab"
          persistent-hint
        >
         <template v-slot:selection="{ item }">
          <!-- HTML that describe how select should render selected items -->
          <div style="display: inline-flex;">
            <div v-for="(color, index) in item.colors[5]"
            :key="index">
              <div :style="patchStyle(color)"></div>
            </div>
          </div>
        </template>
        <template v-slot:item="{ item }">
          <!-- HTML that describe how select should render selected items -->
          <div style="display: inline-flex;">
            <div v-for="(color, index) in item.colors[5]"
            :key="index">
              <div :style="patchStyle(color)"></div>
            </div>
          </div>
        </template>
        </v-select>
          </div>
        <v-simple-table v-if="displayField && displayField.type === 'quantitative'">
          <thead>
          <tr>
            <th><div class=colo></div></th>
            <th>Min</th>
            <th>Max</th>
            <th>Label</th>
          </tr>
        </thead>
          <tbody>
          <tr class="quantitative-bin" v-for="(bin, idx) in bins" :key="idx">
            <td><button :style="patchStyle(bin.color)"></button></td>
            <td>{{bin.min}}</td>
            <td>{{bin.max}}</td>
            <td>
              <v-text-field
              dense
              hide-details="auto"
              label="Label"
              outlined
              v-model="bin.label"
            ></v-text-field>
          </td>
          </tr>
          </tbody>
        <!-- </table> -->
        </v-simple-table>
        <v-simple-table v-if="displayField && displayField.type === 'qualitative'">
          <thead>
          <tr>
            <th><div class=colo></div></th>
            <th>Values</th>
            <th>Label</th>
          </tr>
        </thead>
          <tbody>
          <tr class="quantitative-bin" v-for="(bin, idx) in bins" :key="idx">
            <td><button :style="patchStyle(bin.color)"></button></td>
            <td>{{bin.values}}</td>
            <td>
              <v-text-field
              dense
              hide-details="auto"
              label="Label"
              outlined
              v-model="bin.label"
            ></v-text-field>
          </td>
          </tr>
          </tbody>
        </v-simple-table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ckmeans } from 'simple-statistics';
import COUNTRIES_LOOKUP_JSON from '@/map/country_paths_lookup.json';
import helpers from '@/js/helpers';
import SimpleMap from './SimpleMap.vue';

export default {
  exampleFile: 'data:text/csv;base64,bmFtZSxpZCxjYXRlZ29yaWNhbENvb2xuZXNzLG51bWVyaWNhbENvb2xuZXNzDQpGcmVuY2ggR3VpYW5hLEdVRixOb3QgdmVyeSBDb29sLDI0DQpGcmFuY2UsRlJBLEF2ZXJhZ2UgQ29vbCw5MA0KQWZnaGFuaXN0YW4sQUZHLEF2ZXJhZ2UgQ29vbCwxNw0KQW5nb2xhLEFHTyxBdmVyYWdlIENvb2wsOTMNCkFsYmFuaWEsQUxCLE5vdCB2ZXJ5IENvb2wsMQ0KQW5kb3JyYSxBTkQsTm90IHZlcnkgQ29vbCw1DQpVbml0ZWQgQXJhYiBFbWlyYXRlcyxBUkUsTm90IHZlcnkgQ29vbCw5MA0KQXJnZW50aW5hLEFSRyxBdmVyYWdlIENvb2wsNg0KQXJtZW5pYSxBUk0sTm90IHZlcnkgQ29vbCw5MQ0KRnJlbmNoIFNvdXRoZXJuIFRlcnJpdG9yaWVzLEFURixBdmVyYWdlIENvb2wsNzQNCkF1c3RyYWxpYSxBVVMsQXZlcmFnZSBDb29sLDUzDQpBdXN0cmlhLEFVVCxBdmVyYWdlIENvb2wsODQNCkF6ZXJiYWlqYW4sQVpFLEF2ZXJhZ2UgQ29vbCw5MA0KQnVydW5kaSxCREksQXZlcmFnZSBDb29sLDENCkJlbGdpdW0sQkVMLEF2ZXJhZ2UgQ29vbCw4NQ0KQmVuaW4sQkVOLEF2ZXJhZ2UgQ29vbCw1Mg0KQnVya2luYSBGYXNvLEJGQSxOb3QgdmVyeSBDb29sLDENCkJhbmdsYWRlc2gsQkdELENvb2wsOA0KQnVsZ2FyaWEsQkdSLE5vdCB2ZXJ5IENvb2wsMTANCkJhaGFtYXMsQkhTLEF2ZXJhZ2UgQ29vbCwyNg0KQm9zbmlhIGFuZCBIZXJ6ZWdvdmluYSxCSUgsQXZlcmFnZSBDb29sLDI5DQpCZWxhcnVzLEJMUixBdmVyYWdlIENvb2wsNTINCkJlbGl6ZSxCTFosQXZlcmFnZSBDb29sLDg4DQpCb2xpdmlhIChQbHVyaW5hdGlvbmFsIFN0YXRlIG9mKSxCT0wsQXZlcmFnZSBDb29sLDc5DQpCcmF6aWwsQlJBLEF2ZXJhZ2UgQ29vbCw4NA0KQnJ1bmVpIERhcnVzc2FsYW0sQlJOLEF2ZXJhZ2UgQ29vbCw0OA0KQmh1dGFuLEJUTixBdmVyYWdlIENvb2wsNjQNCkJvdHN3YW5hLEJXQSxBdmVyYWdlIENvb2wsMzENCkNlbnRyYWwgQWZyaWNhbiBSZXB1YmxpYyxDQUYsQXZlcmFnZSBDb29sLDU3DQpDYW5hZGEsQ0FOLENvb2wsMjANClN3aXR6ZXJsYW5kLENIRSxDb29sLDQ4DQpDaGlsZSxDSEwsQXZlcmFnZSBDb29sLDQ1DQpDaGluYSxDSE4sQXZlcmFnZSBDb29sLDc2DQpDw7R0ZSBkJ0l2b2lyZSxDSVYsQXZlcmFnZSBDb29sLDQ2DQpDYW1lcm9vbixDTVIsQXZlcmFnZSBDb29sLDY1DQoiQ29uZ28sIERlbW9jcmF0aWMgUmVwdWJsaWMgb2YgdGhlIixDT0QsQXZlcmFnZSBDb29sLDc0DQpDb25nbyxDT0csQXZlcmFnZSBDb29sLDU0DQpDb2xvbWJpYSxDT0wsQ29vbCw0Ng0KQ29tb3JvcyxDT00sQXZlcmFnZSBDb29sLDYNCkNhYm8gVmVyZGUsQ1BWLENvb2wsNDgNCkNvc3RhIFJpY2EsQ1JJLE5vdCB2ZXJ5IENvb2wsNTUNCkN1YmEsQ1VCLEF2ZXJhZ2UgQ29vbCwyMA0KTm9ydGhlcm4gQ3lwcnVzLENZTixDb29sLDYyDQpDeXBydXMsQ1lQLEF2ZXJhZ2UgQ29vbCw2NQ0KQ3plY2hpYSxDWkUsQXZlcmFnZSBDb29sLDQ5DQpHZXJtYW55LERFVSxDb29sLDkNCkRqaWJvdXRpLERKSSxBdmVyYWdlIENvb2wsMTENCkRlbm1hcmssRE5LLEF2ZXJhZ2UgQ29vbCwzNQ0KRG9taW5pY2FuIFJlcHVibGljLERPTSxBdmVyYWdlIENvb2wsOTYNCkFsZ2VyaWEsRFpBLENvb2wsMjMNCkVjdWFkb3IsRUNVLEF2ZXJhZ2UgQ29vbCwyOQ0KRWd5cHQsRUdZLEF2ZXJhZ2UgQ29vbCw3OQ0KRXJpdHJlYSxFUkksQXZlcmFnZSBDb29sLDQ3DQpTcGFpbixFU1AsQXZlcmFnZSBDb29sLDYwDQpFc3RvbmlhLEVTVCxBdmVyYWdlIENvb2wsNzkNCkV0aGlvcGlhLEVUSCxBdmVyYWdlIENvb2wsMzUNCkZpbmxhbmQsRklOLENvb2wsODcNCkZpamksRkpJLEF2ZXJhZ2UgQ29vbCw2MA0KRmFsa2xhbmQgSXNsYW5kcyAoTWFsdmluYXMpLEZMSyxBdmVyYWdlIENvb2wsODcNCkZhcm9lIElzbGFuZHMsRlJPLEF2ZXJhZ2UgQ29vbCw2OA0KR2Fib24sR0FCLE5vdCB2ZXJ5IENvb2wsMTcNClVuaXRlZCBLaW5nZG9tIG9mIEdyZWF0IEJyaXRhaW4gYW5kIE5vcnRoZXJuIElyZWxhbmQsR0JSLENvb2wsNDUNCkdlb3JnaWEsR0VPLE5vdCB2ZXJ5IENvb2wsMzkNCkdoYW5hLEdIQSxBdmVyYWdlIENvb2wsMTgNCkd1aW5lYSxHSU4sTm90IHZlcnkgQ29vbCw1Mg0KR2FtYmlhLEdNQixBdmVyYWdlIENvb2wsNTENCkd1aW5lYS1CaXNzYXUsR05CLEF2ZXJhZ2UgQ29vbCwyMw0KRXF1YXRvcmlhbCBHdWluZWEsR05RLEF2ZXJhZ2UgQ29vbCw1NQ0KR3JlZWNlLEdSQyxBdmVyYWdlIENvb2wsNTENCkdyZWVubGFuZCxHUkwsQXZlcmFnZSBDb29sLDc0DQpHdWF0ZW1hbGEsR1RNLEF2ZXJhZ2UgQ29vbCw0Mg0KR3V5YW5hLEdVWSxBdmVyYWdlIENvb2wsMjgNCkhvbmcgS29uZyxIS0csQXZlcmFnZSBDb29sLDM0DQpIb25kdXJhcyxITkQsQXZlcmFnZSBDb29sLDI4DQpDcm9hdGlhLEhSVixBdmVyYWdlIENvb2wsNQ0KSGFpdGksSFRJLENvb2wsMzANCkh1bmdhcnksSFVOLEF2ZXJhZ2UgQ29vbCw1Mg0KSW5kb25lc2lhLElETixDb29sLDU4DQpJbmRpYSxJTkQsQ29vbCwzOA0KSXJlbGFuZCxJUkwsQXZlcmFnZSBDb29sLDExDQpJcmFuIChJc2xhbWljIFJlcHVibGljIG9mKSxJUk4sQXZlcmFnZSBDb29sLDMzDQpJcmFxLElSUSxBdmVyYWdlIENvb2wsMTYNCkljZWxhbmQsSVNMLEF2ZXJhZ2UgQ29vbCw0MQ0KSXNyYWVsLElTUixBdmVyYWdlIENvb2wsMTQNCkl0YWx5LElUQSxBdmVyYWdlIENvb2wsMjgNCkphbWFpY2EsSkFNLEF2ZXJhZ2UgQ29vbCw0Mw0KSm9yZGFuLEpPUixBdmVyYWdlIENvb2wsMzYNCkphcGFuLEpQTixBdmVyYWdlIENvb2wsNA0KS2FzaG1pcixLQVMsQ29vbCwxOQ0KS2F6YWtoc3RhbixLQVosQ29vbCw1MQ0KS2VueWEsS0VOLENvb2wsNTkNCkt5cmd5enN0YW4sS0daLE5vdCB2ZXJ5IENvb2wsNDMNCkNhbWJvZGlhLEtITSxBdmVyYWdlIENvb2wsOTINCiJLb3JlYSwgUmVwdWJsaWMgb2YiLEtPUixBdmVyYWdlIENvb2wsNDYNCktvc292byxLT1MsTm90IHZlcnkgQ29vbCw0NQ0KS3V3YWl0LEtXVCxBdmVyYWdlIENvb2wsMjUNCkxhbyBQZW9wbGUncyBEZW1vY3JhdGljIFJlcHVibGljLExBTyxBdmVyYWdlIENvb2wsNjANCkxlYmFub24sTEJOLENvb2wsNjUNCkxpYmVyaWEsTEJSLEF2ZXJhZ2UgQ29vbCwzMQ0KTGlieWEsTEJZLE5vdCB2ZXJ5IENvb2wsNzANCkxpZWNodGVuc3RlaW4sTElFLEF2ZXJhZ2UgQ29vbCwxDQpTcmkgTGFua2EsTEtBLEF2ZXJhZ2UgQ29vbCw2NA0KTGVzb3RobyxMU08sQXZlcmFnZSBDb29sLDcNCkxpdGh1YW5pYSxMVFUsQXZlcmFnZSBDb29sLDE2DQpMdXhlbWJvdXJnLExVWCxDb29sLDkzDQpMYXR2aWEsTFZBLEF2ZXJhZ2UgQ29vbCw0NQ0KTWFjYW8sTUFDLEF2ZXJhZ2UgQ29vbCw3Nw0KU2FpbnQgTWFydGluIChGcmVuY2ggcGFydCksTUFGLE5vdCB2ZXJ5IENvb2wsNDUNCk1vcm9jY28sTUFSLENvb2wsMjINCk1vbmFjbyxNQ08sQXZlcmFnZSBDb29sLDkyDQoiTW9sZG92YSwgUmVwdWJsaWMgb2YiLE1EQSxBdmVyYWdlIENvb2wsOTQNCk1hZGFnYXNjYXIsTURHLENvb2wsMTANCk1leGljbyxNRVgsQ29vbCw2OA0KTm9ydGggTWFjZWRvbmlhLE1LRCxBdmVyYWdlIENvb2wsNjMNCk1hbGksTUxJLEF2ZXJhZ2UgQ29vbCw2MA0KTXlhbm1hcixNTVIsQ29vbCw3Ng0KTW9udGVuZWdybyxNTkUsQXZlcmFnZSBDb29sLDU2DQpNb25nb2xpYSxNTkcsQXZlcmFnZSBDb29sLDgwDQpNb3phbWJpcXVlLE1PWixOb3QgdmVyeSBDb29sLDINCk1hdXJpdGFuaWEsTVJULE5vdCB2ZXJ5IENvb2wsNTANCk1hdXJpdGl1cyxNVVMsQXZlcmFnZSBDb29sLDc1DQpNYWxhd2ksTVdJLEF2ZXJhZ2UgQ29vbCwzMw0KTWFsYXlzaWEsTVlTLEF2ZXJhZ2UgQ29vbCw4Ng0KTmFtaWJpYSxOQU0sQXZlcmFnZSBDb29sLDYyDQpOZXcgQ2FsZWRvbmlhLE5DTCxOb3QgdmVyeSBDb29sLDc3DQpOaWdlcixORVIsQXZlcmFnZSBDb29sLDcyDQpOaWdlcmlhLE5HQSxOb3QgdmVyeSBDb29sLDU2DQpOaWNhcmFndWEsTklDLEF2ZXJhZ2UgQ29vbCw1MA0KTmV0aGVybGFuZHMsTkxELEF2ZXJhZ2UgQ29vbCwyDQpOb3J3YXksTk9SLEF2ZXJhZ2UgQ29vbCw1Mg0KTmVwYWwsTlBMLENvb2wsNDINCk5ldyBaZWFsYW5kLE5aTCxBdmVyYWdlIENvb2wsMzcNCk9tYW4sT01OLEF2ZXJhZ2UgQ29vbCw3NQ0KUGFraXN0YW4sUEFLLEF2ZXJhZ2UgQ29vbCw4Mg0KUGFuYW1hLFBBTixBdmVyYWdlIENvb2wsNjQNClBlcnUsUEVSLEF2ZXJhZ2UgQ29vbCwzOQ0KUGhpbGlwcGluZXMsUEhMLEF2ZXJhZ2UgQ29vbCwzMA0KUGFwdWEgTmV3IEd1aW5lYSxQTkcsSWNlIENvbGQsMjUNClBvbGFuZCxQT0wsTm90IHZlcnkgQ29vbCwyMw0KUHVlcnRvIFJpY28sUFJJLEF2ZXJhZ2UgQ29vbCw2Ng0KS29yZWEgKERlbW9jcmF0aWMgUGVvcGxlJ3MgUmVwdWJsaWMgb2YpLFBSSyxBdmVyYWdlIENvb2wsMjMNClBvcnR1Z2FsLFBSVCxBdmVyYWdlIENvb2wsNzMNClBhcmFndWF5LFBSWSxDb29sLDQ1DQpQYWxlc3RpbmlhbiBUZXJyaXRvcmllcyxQU1gsTm90IHZlcnkgQ29vbCwxNg0KUWF0YXIsUUFULEF2ZXJhZ2UgQ29vbCw1Mw0KUm9tYW5pYSxST1UsQ29vbCw3DQpSdXNzaWFuIEZlZGVyYXRpb24sUlVTLEF2ZXJhZ2UgQ29vbCwxMw0KUndhbmRhLFJXQSxDb29sLDk0DQpXZXN0ZXJuIFNhaGFyYSxTQUgsQ29vbCw3NQ0KU2F1ZGkgQXJhYmlhLFNBVSxBdmVyYWdlIENvb2wsODUNClN1ZGFuLFNETixDb29sLDM3DQpTb3V0aCBTdWRhbixTU0QsQ29vbCwzNQ0KU2VuZWdhbCxTRU4sQXZlcmFnZSBDb29sLDY5DQpTb3V0aCBHZW9yZ2lhIGFuZCB0aGUgU291dGggU2FuZHdpY2ggSXNsYW5kcyxTR1MsQXZlcmFnZSBDb29sLDk3DQpTb2xvbW9uIElzbGFuZHMsU0xCLE5vdCB2ZXJ5IENvb2wsNDANClNpZXJyYSBMZW9uZSxTTEUsQXZlcmFnZSBDb29sLDMNCkVsIFNhbHZhZG9yLFNMVixBdmVyYWdlIENvb2wsOTYNClNhbiBNYXJpbm8sU01SLEF2ZXJhZ2UgQ29vbCwzMQ0KU2VyYmlhLFNSQixBdmVyYWdlIENvb2wsNDINClN1cmluYW1lLFNVUixBdmVyYWdlIENvb2wsOTANClNsb3Zha2lhLFNWSyxDb29sLDY0DQpTbG92ZW5pYSxTVk4sQXZlcmFnZSBDb29sLDQ4DQpTd2VkZW4sU1dFLEF2ZXJhZ2UgQ29vbCwyNA0KRXN3YXRpbmksU1daLEF2ZXJhZ2UgQ29vbCw0MQ0KU2ludCBNYWFydGVuIChEdXRjaCBwYXJ0KSxTWE0sQXZlcmFnZSBDb29sLDM5DQpTeXJpYW4gQXJhYiBSZXB1YmxpYyxTWVIsQXZlcmFnZSBDb29sLDE2DQpDaGFkLFRDRCxBdmVyYWdlIENvb2wsMTYNClRvZ28sVEdPLEF2ZXJhZ2UgQ29vbCw0DQpUaGFpbGFuZCxUSEEsQ29vbCw2Mw0KVGFqaWtpc3RhbixUSkssQXZlcmFnZSBDb29sLDI0DQpUdXJrbWVuaXN0YW4sVEtNLEljZSBDb2xkLDUyDQpUaW1vci1MZXN0ZSxUTFMsSWNlIENvbGQsOQ0KVHJpbmlkYWQgYW5kIFRvYmFnbyxUVE8sQXZlcmFnZSBDb29sLDk3DQpUdW5pc2lhLFRVTixBdmVyYWdlIENvb2wsMzUNClR1cmtleSxUVVIsQXZlcmFnZSBDb29sLDg4DQoiVGFpd2FuLCBQcm92aW5jZSBvZiBDaGluYSIsVFdOLE5vdCB2ZXJ5IENvb2wsNzcNCiJUYW56YW5pYSwgVW5pdGVkIFJlcHVibGljIG9mIixUWkEsQ29vbCw4Mw0KVWdhbmRhLFVHQSxBdmVyYWdlIENvb2wsMQ0KVWtyYWluZSxVS1IsQXZlcmFnZSBDb29sLDgwDQpVcnVndWF5LFVSWSxOb3QgdmVyeSBDb29sLDczDQpVbml0ZWQgU3RhdGVzIG9mIEFtZXJpY2EsVVNBLEF2ZXJhZ2UgQ29vbCwyNA0KVXpiZWtpc3RhbixVWkIsTm90IHZlcnkgQ29vbCw1NQ0KVmVuZXp1ZWxhIChCb2xpdmFyaWFuIFJlcHVibGljIG9mKSxWRU4sQXZlcmFnZSBDb29sLDEwDQpWaWV0IE5hbSxWTk0sQ29vbCw0MQ0KVmFudWF0dSxWVVQsTm90IHZlcnkgQ29vbCw0NA0KU2Ftb2EsV1NNLE5vdCB2ZXJ5IENvb2wsMTINClllbWVuLFlFTSxBdmVyYWdlIENvb2wsMjUNClNvdXRoIEFmcmljYSxaQUYsQXZlcmFnZSBDb29sLDUxDQpaYW1iaWEsWk1CLEF2ZXJhZ2UgQ29vbCw2Nw0KWmltYmFid2UsWldFLEF2ZXJhZ2UgQ29vbCw0OA0KU29tYWxpYSxTT00sQXZlcmFnZSBDb29sLDU0DQo=',
  components: { SimpleMap },
  name: 'MapMaker',
  countriesData: COUNTRIES_LOOKUP_JSON,
  props: {
    msg: String,
  },
  data() {
    return {
      dataExplorerTab: 'data-source',
      symbologyTab: 'general-color',
      file: null,
      readFile: null,
      activeColor: 'background',
      activeLegendColor: 'background',
      activeBin: 0,
      legend: {
        colors: {
          background: {
            color: {
              r: 255,
              g: 255,
              b: 255,
              a: 0.3,
            },
            name: 'Background',
            key: 'background',
          },
        },
        patch: {
          width: 25,
          height: 25,
        },
      },
      colors: {
        background: { color: '#0A0A0A', name: 'Background', key: 'background' },
        noData: { color: '#27251b', name: 'No Data', key: 'noData' },
        primaryStroke: { color: '#111111', name: 'Primary Stroke', key: 'primaryStroke' },
        focus: { color: '#ADACAA', name: 'Focus/Hover', key: 'focus' },
      },
      rawStrokeWidth: 20,
      colorSchemesType: ['sequential'],
      colorSchemesFamily: 0,
      activeColorScheme:
        {
          name: 'Reds',
          colors: {
            3: ['#fee8c8', '#fdbb84', '#e34a33'],
            4: ['#fef0d9', '#fdcc8a', '#fc8d59', '#d7301f'],
            5: ['#fef0d9', '#fdcc8a', '#fc8d59', '#e34a33', '#b30000'],
            6: ['#fef0d9', '#fdd49e', '#fdbb84', '#fc8d59', '#e34a33', '#b30000'],
          },
        },
      colorSchemes: {
        sequential: [
          {
            name: 'Greens',
            colors: {
              3: ['#e5f5f9', '#99d8c9', '#2ca25f'],
              4: ['#edf8fb', '#b2e2e2', '#66c2a4', '#238b45'],
              5: ['#edf8fb', '#b2e2e2', '#66c2a4', '#2ca25f', '#006d2c'],
              6: ['#edf8fb', '#ccece6', '#99d8c9', '#66c2a4', '#2ca25f', '#006d2c'],
            },
          },
          {
            name: 'Reds',
            colors: {
              3: ['#fee8c8', '#fdbb84', '#e34a33'],
              4: ['#fef0d9', '#fdcc8a', '#fc8d59', '#d7301f'],
              5: ['#fef0d9', '#fdcc8a', '#fc8d59', '#e34a33', '#b30000'],
              6: ['#fef0d9', '#fdd49e', '#fdbb84', '#fc8d59', '#e34a33', '#b30000'],
            },
          },
          {
            name: 'Blues',
            colors: {
              3: ['#ece7f2', '#a6bddb', '#2b8cbe'],
              4: ['#f1eef6', '#bdc9e1', '#74a9cf', '#0570b0'],
              5: ['#f1eef6', '#bdc9e1', '#74a9cf', '#2b8cbe', '#045a8d'],
              6: ['#f1eef6', '#d0d1e6', '#a6bddb', '#74a9cf', '#2b8cbe', '#045a8d'],
            },
          },
        ],
        divergent: [
          {
            name: '',
            colors: {
              3: ['#fc8d59', '#ffffbf', '#91bfdb'],
              4: ['#d7191c', '#fdae61', '#abd9e9', '#2c7bb6'],
              5: ['#d7191c', '#fdae61', '#ffffbf', '#abd9e9', '#2c7bb6'],
              6: ['#d73027', '#fc8d59', '#fee090', '#e0f3f8', '#91bfdb', '#4575b4'],
              7: ['#d73027', '#fc8d59', '#fee090', '#ffffbf', '#e0f3f8', '#91bfdb', '#4575b4'],
              8: ['#d73027', '#f46d43', '#fdae61', '#fee090', '#e0f3f8', '#abd9e9', '#74add1', '#4575b4'],
            },
          },
        ],
        qualitative: [
          {
            name: '',
            colors: {
              3: ['#7fc97f', '#beaed4', '#fdc086'],
              4: ['#7fc97f', '#beaed4', '#fdc086', '#ffff99'],
              5: ['#7fc97f', '#beaed4', '#fdc086', '#ffff99', '#386cb0'],
              6: ['#7fc97f', '#beaed4', '#fdc086', '#ffff99', '#386cb0', '#f0027f'],
              7: ['#7fc97f', '#beaed4', '#fdc086', '#ffff99', '#386cb0', '#f0027f', '#bf5b17'],
              8: ['#7fc97f', '#beaed4', '#fdc086', '#ffff99', '#386cb0', '#f0027f', '#bf5b17', '#666666'],
            },
          },
          {
            name: '',
            colors: {
              3: ['#e41a1c', '#377eb8', '#4daf4a'],
              4: ['#e41a1c', '#377eb8', '#4daf4a', '#984ea3'],
              5: ['#e41a1c', '#377eb8', '#4daf4a', '#984ea3', '#ff7f00'],
              6: ['#e41a1c', '#377eb8', '#4daf4a', '#984ea3', '#ff7f00', '#ffff33'],
              7: ['#e41a1c', '#377eb8', '#4daf4a', '#984ea3', '#ff7f00', '#ffff33', '#a65628'],
              8: ['#e41a1c', '#377eb8', '#4daf4a', '#984ea3', '#ff7f00', '#ffff33', '#a65628', '#f781bf'],
            },
          },
          {
            name: '',
            colors: {
              3: ['#b3e2cd', '#fdcdac', '#cbd5e8'],
              4: ['#b3e2cd', '#fdcdac', '#cbd5e8', '#f4cae4'],
              5: ['#b3e2cd', '#fdcdac', '#cbd5e8', '#f4cae4', '#e6f5c9'],
              6: ['#b3e2cd', '#fdcdac', '#cbd5e8', '#f4cae4', '#e6f5c9', '#fff2ae'],
              7: ['#b3e2cd', '#fdcdac', '#cbd5e8', '#f4cae4', '#e6f5c9', '#fff2ae', '#f1e2cc'],
              8: ['#b3e2cd', '#fdcdac', '#cbd5e8', '#f4cae4', '#e6f5c9', '#fff2ae', '#f1e2cc', '#cccccc'],
            },
          },
        ],
      },
      dataFields: [],
      dataRows: [],
      symbologyType: null,
      displayField: null,
      bins: [],
      maxSymbols: 5,
    };
  },
  computed: {
    allColorSchemes() {
      const colorSchemes = [];
      colorSchemes.push(...this.colorSchemes.sequential);
      colorSchemes.push(...this.colorSchemes.divergent);
      colorSchemes.push(...this.colorSchemes.qualitative);
      return colorSchemes;
    },
    primaryStrokeWidth() {
      return this.rawStrokeWidth / 100;
    },
    items() {
      const numFields = this.dataFields.length;
      const rows = this.dataRows.filter((row) => row.length === numFields);
      return rows.map((element) => helpers.rowToObject(element, this.dataFields));
    },
    fields() {
      let fields = [];
      const excludeFields = ['name', 'id'];
      // get fields to check
      for (const property of this.dataFields) { // eslint-disable-line
        if (!excludeFields.includes(property)) {
          fields.push(property);
        }
      }
      fields = fields.map((field) => {
        let fieldType = null;
        let text = '';
        if (this.isQuantitative(field, this.items)) {
          fieldType = 'quantitative';
          text = `${field} (quantitative)`;
        } else {
          fieldType = 'qualitative';
          text = `${field} (qualitative)`;
        }
        return { name: field, type: fieldType, text };
      });
      // const quantitativeFields = fields.filter((field) =>
      // this.isQuantitative(field, this.items));
      // const qualitativeFields = fields.filter((field) => !quantitativeFields.includes(field));
      return fields;
    },
    settings() {
      return {
        bins: this.bins,
        colors: this.colors,
        legend: this.legend,
        displayField: this.displayField,
        primaryStrokeWidth: this.primaryStrokeWidth,
      };
    },
  },
  watch: {
    settings: {
      handler() {
        if (this.$refs.map) {
          this.$refs.map.initMap();
        }
      },
      deep: true,
    },
    file(newval) {
      this.loadFile(newval);
    },
    displayField() {
      this.refreshSettings();
    },
    fields(newVal) {
      // When fields change ie load, the default active field should be the first quantitative one
      let activeField = null;
      if (newVal.length > 0) {
        activeField = newVal[0]; // eslint-disable-line
      }
      newVal.some((field) => {
        if (field.type === 'quantitative') {
          activeField = field;
          return true;
        }
        return false;
      });
      this.displayField = activeField;
    },
    backgroundColor(newval, oldval) {
      const rx = /^#([0-9a-f]{3}|[0-9a-f]{6})$/i;
      if (!rx.test(newval)) {
        this.backgroundColor = oldval;
      }
    },
  },
  methods: {
    refreshSettings() {
      if (this.displayField.type === 'quantitative') {
        const values = this.items.map((item) => item[this.displayField.name]);
        this.generateQuantitativeBins(values, this.maxSymbols);
      } else {
        this.generateQualitativeBins(this.items.map((item) => item[this.displayField.name]), 5);
      }
    },
    downloadExample() {
      const a = document.createElement('a');
      // a.href = window.URL.createObjectURL(xhr.response);
      a.href = this.$options.exampleFile;
      a.download = 'example.csv';
      a.style.display = 'none';
      document.body.appendChild(a);
      a.click();
      a.remove();
      // const newTab = window.open('about:blank', 'your hitomezashi');
      // newTab.document.write(`<img src="${dataURL}" alt="from canvas"/>`);
    },
    patchStyle(color) {
      return {
        border: '1px solid black',
        'background-color': color,
        width: '25px',
        height: '25px',
      };
    },
    b64ToFile(input) {
      const url = input;
      fetch(url)
        .then((res) => res.blob())
        .then((blob) => {
          this.file = new File([blob], 'Example.csv', { type: 'text/csv' });
        });
    },
    loadFile(file) {
      const reader = new FileReader();

      reader.addEventListener('load', () => {
        // convert image file to base64 string
        const csvResult = helpers.CSVToArray(reader.result);
        this.dataFields = csvResult.shift();
        this.dataRows = csvResult;
      }, false);

      if (file) {
        reader.readAsText(file);
        this.readFile = reader.result;
      }
    },
    isQuantitative(property, rows) {
      if (rows.every((row) => Number(row[property]))) {
        return true;
      }
      return false;
    },
    generateQualitativeBins(values, maxFields) {
      const fields = [...new Set(values)];
      let bins = [];
      for (let i = 0; i < fields.length; i += 1) {
        const binIndex = i % maxFields;
        if (bins.length <= binIndex) {
          bins.push([]);
        }
        bins[binIndex].push(fields[i]);
      }
      bins = bins.map((bin, nth) => (
        {
          values: bin,
          color: this.activeColorScheme.colors[bins.length][nth], // eslint-disable-line
          label: `${bin.join(',')}`,
        }
      ));
      this.bins.splice(0, this.bins.length, ...bins);
    },
    generateQuantitativeBins(data, n) {
      const bins = ckmeans(data, n);
      const newBins = bins.map((bin, index) => {
        const newObj = {};
        newObj.min = bin.shift();
        newObj.max = bin.pop();
        newObj.color = this.activeColorScheme.colors[n][index]; // eslint-disable-line
        newObj.label = `${newObj.min} - ${newObj.max}`;
        return newObj;
      });
      this.bins.splice(0, this.bins.length);
      newBins.forEach((bin, index) => {
        const newObject = Object.assign({}, bin); // eslint-disable-line
        this.$set(this.bins, index, newObject);
      });
    },
    loadExampleCSV() {
      this.b64ToFile(this.$options.exampleFile);
    },
  },
  mounted() {
    // helpers.test.foo();
    this.loadExampleCSV();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.main-wrapper {
  margin-top: 2em;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 10px;
  max-width: 100vw;
}

.map {
  grid-column: 1 / 3;
}

.data-explorer {
  grid-column: 1 / 4;
  max-width: 1000px;
  margin: auto;
}

.country {
    fill: #cccccc;
}

.color-controls {
  display: grid;
  grid-template-columns: auto auto;
  align-items: center;
  justify-items: start;
  justify-content: center;
  gap: 10px;
  margin: 10px
}

.color-control {
  cursor: pointer;
  display: grid;
  grid-template-columns: repeat(2, auto);
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.color-control-label {
  cursor: pointer;
}
.color-control-label.active {
  color: blue;
}

.patch-control {
  display: flex;
  align-items: center;
}
.patch-control .v-text-field {
  max-width: 80px;
}

/* .color-picker {
  margin: auto;
} */
.v-color-picker {
  margin: auto;
}

.classification-tab {
  display:  grid;
  grid-template-columns: 1fr 1fr;
  justify-items: center;
}

.classification-table {
  margin: auto;
}

.tabs-menu {
  display: flex;
  border-bottom: 1px solid #cccccc;
  margin-bottom: 2em;
}
.tabs {
  margin: auto;
}

.tab {
  padding-left: .5em;
  padding-right: .5em;
}

.tab.active-tab {
  border-bottom: 1px solid black;
}

.file-input {
  margin: 0 1em 0 1em;
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}

@media only screen and (max-width: 1100px) {
  .main-wrapper {
    grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
    align-items: start;
  }
  .map {
      grid-column: 1 / 3;
    }

  .data-explorer {
    grid-column: 2 / 3;
    max-width: 1000px;
    margin: 0;
  }

  .classification-tab {
    grid-template-columns: minmax(0, 1fr);
  }

}

@media only screen and (max-width: 600px) {
  .main-wrapper {
    grid-template-columns: minmax(0, 1fr);
    align-items: start;
  }
  .map {
      grid-column: 1;
    }

  .data-explorer {
    grid-column: 1;
    max-width: 1000px;
    margin: 0;
  }

}

</style>
