<template>
    <div class="w-100" ref="map-root" style="width: 100%; height: 100%">
    </div>
</template>

<style>
html,
body {
    height: 100%;
    margin: 0;
}

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    height: 100%;
    width: 100vh;
    display: grid;
    grid-template-columns: 100vh;
    grid-auto-rows: 1fr;
    grid-gap: 1rem;

    box-sizing: border-box;
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
    methods: {

        createCoordinat(models) {
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
                let longitude = item.geolocation.longitude || 0.0
                let latitude = item.geolocation.latitude || 0.0
                this.center =[longitude, latitude]
                var coord = [longitude, latitude]
                this.data.geometry.coordinates[0].push(coord)
            })

            return this.data

        },

        async getCoordinat() {
            const response = await fetch(api)
            const models = await response.json()
            return this.createCoordinat(models)
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
                    zoom: 3,
                    center: this.center,
                    constrainResolution: true
                }),
            })
        }
    },
    mounted() {
        this.createMap()
    },
}
</script>