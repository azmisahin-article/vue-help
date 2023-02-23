<template>
    <div id="map" class="w-100" ref="map-root">
    </div>
</template>

<style>
#map {
    width: 100%;
    height: 100%;
    min-width: 100vh;
    min-height: 80vh;
}
</style>
  
<script>

import View from 'ol/View'
import Map from 'ol/Map'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'

// we’ll need these additional imports
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import GeoJSON from 'ol/format/GeoJSON'

// importing the OpenLayers stylesheet is required for having
// good looking buttons!
import 'ol/ol.css'
import { api } from './helpers'

export default {
    name: 'MapContainer',
    components: {},
    props: {},
    data: () => ({
        //api endpoint
        center: [30.6409619, 36.9359715]
    }),
    methods: {

        async createCoordinat(models) {
            this.data = {
                type: 'Feature',
                properties: {},
                geometry: {
                    type: 'Polygon',
                    coordinates: [
                        []
                    ]
                }
            };
            models.forEach((item) => {
                let longitude = item.longitude || 0.0
                let latitude = item.latitude || 0.0
                this.center = [longitude, latitude]
                var coord = [longitude, latitude]
                this.data.geometry.coordinates[0].push(coord)
            })

            return await this.data

        },

        async getCoordinat() {
            const response = await fetch(`${api}/geolocation`)
            const models = await response.json()
            return await this.createCoordinat(models)
        },
        async createMap() {
            let data = await this.getCoordinat()

            // a feature (geospatial object) is created from the GeoJSON
            const feature = new GeoJSON().readFeature(data, {
                // this is required since GeoJSON uses latitude/longitude,
                // but the map is rendered using “Web Mercator”
                featureProjection: 'EPSG:3857'
            });

            // a new vector layer is created with the feature
            const vectorLayer = new VectorLayer({
                source: new VectorSource({
                    features: [feature],
                }),
            })

            // this is where we create the OpenLayers map
            new Map({
                // the map will be created using the 'map-root' ref
                target: this.$refs['map-root'],
                layers: [
                    // adding a background tiled layer
                    new TileLayer({
                        source: new OSM() // tiles are served by OpenStreetMap
                    }),
                    // the vector layer is added above the tiled OSM layer
                    vectorLayer
                ],

                // the map view will initially show the whole world
                view: new View({
                    zoom: 7,
                    center: this.center,
                    constrainResolution: true
                }),
            })
        }
    },
    async mounted() {
        await this.createMap()
    },
}
</script>