<template>
  <v-dialog scrollable persistent>
    <v-card>
      <v-card-title>
        {{ $t('LocationSelector.Title') }} <v-btn style="float:right;" variant="text" density="compact" icon="mdi-close" @click="close"></v-btn>
      </v-card-title>

      <v-divider></v-divider>

      <v-card-text>
        <v-form @submit.prevent="search">
          <v-text-field
            ref="locationInput"
            v-model="locationSearchForm.q"
            :label="$t('LocationSelector.SearchByName')"
            type="text"
            append-inner-icon="mdi-magnify"
            @click:append-inner="search"
            :rules="[fieldRequired]"
            hide-details="auto"
            :loading="loading"
            required>
          </v-text-field>
        </v-form>

        <p v-if="searchProvider === 'nominatim'" class="text-caption text-warning mt-2">
          <i18n-t keypath="LocationSelector.Warning" tag="i">
            <template #newline><br /></template>
          </i18n-t>
        </p>

        <v-sheet v-if="results && Array.isArray(results)">
          <v-divider></v-divider>

          <h3>
            <i18n-t keypath="LocationSelector.Result" tag="span">
              <template v-slot:resultNumber>
                <small>{{ results.length }}</small>
              </template>
            </i18n-t>
          </h3>
          <v-row>
            <v-col cols="12" sm="6">
              <v-card
                class="mb-2"
                width="100%"
                v-for="location in results"
                elevation="1"
                @click="selectLocation(location)">
                <v-card-text>
                  <h4>{{ getLocationTitle(location, true, false, false) }}</h4>
                  {{ getLocationTitle(location, false, true, true) }}<br />
                  <v-chip label size="small" density="comfortable">{{ getLocationCategory(location) }}</v-chip>
                </v-card-text>
              </v-card>
            </v-col>
            <v-col cols="12" sm="6" style="min-height:200px">
              <l-map ref="map" v-model:zoom="mapZoom" :center="mapCenter" :use-global-leaflet="false" @ready="initMap">
                <l-tile-layer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" layer-type="base" name="OpenStreetMap"></l-tile-layer>
                <l-marker v-for="location in results" :lat-lng="getLocationLatLng(location)">
                  <l-popup>
                    <h4>{{ getLocationTitle(location, true, false, false) }}</h4>
                    {{ getLocationTitle(location, false, true, true) }}<br />
                    <v-chip label size="small" density="comfortable">{{ getLocationCategory(location) }}</v-chip>
                  </l-popup>
                </l-marker>
              </l-map>
            </v-col>
          </v-row>
        </v-sheet>

        <v-sheet v-if="results && (typeof results === 'string')">{{ results }}</v-sheet>

        <v-sheet v-if="recentLocations.length">
          <v-divider class="mt-2 mb-2"></v-divider>

          <h3 class="mb-1">
            <i18n-t keypath="LocationSelector.RecentLocations" tag="span">
              <template v-slot:recentLocationNumber>
                <small>{{ recentLocations.length }}</small>
              </template>
            </i18n-t>
          </h3>
          <v-chip
            class="mb-2"
            closable
            v-for="location in recentLocations"
            :key="getLocationUniqueID(location)"
            prepend-icon="mdi-history"
            close-icon="mdi-delete"
            @click="selectLocation(location)"
            @click:close="removeRecentLocation(location)">
            {{ getLocationTitle(location, true, true, true) }}
          </v-chip>
          <br />
          <v-btn size="small" @click="clearRecentLocations">
            {{ $t('LocationSelector.Clear') }} </v-btn>
        </v-sheet>
      </v-card-text>

      <v-divider></v-divider>

      <v-card-actions class="justify-end">
        <div>
          <i18n-t keypath="LocationSelector.PoweredBy.text" tag="span">
            <template #url>
              <a v-if="searchProvider === 'nominatim'" href="https://nominatim.openstreetmap.org" target="_blank">Nominatim (OpenStreetMap)</a>
              <a v-if="searchProvider === 'photon'" href="https://photon.komoot.io" target="_blank">Komoot Photon (OpenStreetMap)</a>
            </template>
          </i18n-t>
        </div>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import 'leaflet/dist/leaflet.css'
import { LMap, LTileLayer, LMarker, LPopup } from '@vue-leaflet/vue-leaflet'
import { mapStores } from 'pinia'
import { useAppStore } from '../store'
import api from '../services/api'
import utils from '../utils.js'

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
  },
  data() {
    return {
      // location form
      locationSearchForm: {
        q: ''
      },
      loading: false,
      results: null,
      // map
      map: null,
      mapZoom: 5,
      mapCenter: [45, 5],
      mapBounds: null,
      // search
      searchProvider: 'photon',  // 'nominatim', 'photon'
    }
  },
  computed: {
    ...mapStores(useAppStore),
    formFilled() {
      return Object.values(this.locationSearchForm).every(x => !!x)
    },
    recentLocations() {
      return this.appStore.getRecentLocations()
    },
  },
  mounted() {
    this.$refs.locationInput.focus()
  },
  methods: {
    fieldRequired(v) {
      return !!v
    },
    initMap() {  // TODO: improve map setup
      this.map = this.$refs.map.leafletObject
      if (this.mapBounds) {
        this.map.fitBounds(this.mapBounds)
      }
    },
    search() {
      this.results = null
      this.loading = true
      api.openstreetmapSearch(this.locationSearchForm.q, this.searchProvider)
      .then((data) => {
        this.loading = false
        if (data.length) {
          this.results = data
          if (this.results.length > 1) {
            this.mapBounds = utils.getMapBounds(this.results, this.searchProvider)
          } else {
            this.mapCenter = utils.getMapCenter(this.results, this.searchProvider)
            this.mapZoom = 12
            this.mapBounds = null
          }
        } else {
          this.results = this.$t('LocationSelector.NoResult')
        }
      })
    },
    getLocationTitle(location, withName=true, withRoad=false, withCity=true) {
      return utils.getLocationTitle(location, withName, withRoad, withCity)
    },
    getLocationUniqueID(location) {
      return utils.getLocationUniqueID(location)
    },
    getLocationCategory(location) {
      return utils.getLocationCategory(location)
    },
    getLocationLatLng(location) {
      return utils.getLocationLatLng(location)
    },
    selectLocation(location) {
      this.$emit('location', location)
      this.close()
    },
    removeRecentLocation(location) {
      this.appStore.removeRecentLocation(location)
    },
    clearRecentLocations() {
      this.appStore.clearRecentLocations()
    },
    close() {
      this.$emit('close')
    },
  },
}
</script>
