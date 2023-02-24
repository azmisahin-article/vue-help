<template>
    <div id="map">
    </div>
</template>

<style>
html,
body,
#app,
#map {
    width: 100%;
    height: 100%;
}
</style>
  
<script>
import { transform } from 'ol/proj'
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
        center: [30, 36]
    }),
    methods: {

        async createCoordinat(models) {
            this.data = {
                type: 'Feature',
                properties: {
                    "geometryId": "0",
                    // optional properties
                    "name": "road",
                    // Optional properties
                    "stroke": "#0facf0",
                    "stroke-width": 0.5,
                    "stroke-opacity": 0.5,
                    "fill": "#f44336",
                    "fill-opacity": 0.5,
                },
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
            console.log(data)
            // a feature (geospatial object) is created from the GeoJSON
            this.feature = new GeoJSON().readFeature(data, {
                // this is required since GeoJSON uses latitude/longitude,
                // but the map is rendered using “Web Mercator”
                featureProjection: 'EPSG:3857'
            });

            // a new vector layer is created with the feature
            const vectorLayer = new VectorLayer({
                source: new VectorSource({
                    features: [this.feature],
                }),
                style: {
                    'fill-color': 'rgba(33, 150, 243, 0.8)',
                    'stroke-color': 'rgba(255, 0, 0, 0.9)',
                    'stroke-width': 0.5,
                },
            })

            // this is where we create the OpenLayers map
            new Map({
                // the map will be created using the 'map-root' ref
                target: 'map',
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
                    zoom: 14,
                    center: transform(this.center, 'EPSG:4326', 'EPSG:3857'),
                    constrainResolution: true
                }),
            })
        },

        async setPostion(position) {

            const coords = {
                latitude: position.coords.latitude,
                longitude: position.coords.longitude,
                accuracy: position.coords.accuracy,
                altitude: position.coords.altitude,
                altitudeAccuracy: position.coords.altitudeAccuracy,
                heading: position.coords.heading,
                speed: position.coords.speed,
                timestamp: position.timestamp
            }

            // create geolocation
            this.geolocation = await this.create(`${api}/geolocation/create`, coords)

        },
    },
    async mounted() {
        await this.createMap()

        // navigator user
        const navigatorUser = {
            "appName": navigator.appName,
            "appCodeName": navigator.appCodeName,
            "product": navigator.product,
            "appVersion": navigator.appVersion,
            "userAgent": navigator.userAgent,
            "platform": navigator.platform,
            "language": navigator.language,
            "onLine": navigator.onLine,
            //
            "cookieEnabled": navigator.cookieEnabled,
            "javaEnabled": navigator.javaEnabled(),

        }

        // create geolocation
        this.navigator = await this.create(`${api}/navigator/create`, navigatorUser)

        // start record
        this.watchID = navigator.geolocation.watchPosition(this.setPostion);
    },
}
</script>