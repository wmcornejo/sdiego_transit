<template>
  <div id="map"></div>
</template>

<script setup>
import { onMounted } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

onMounted(() => {

  const mapElement = document.getElementById('map')
  const map = L.map(mapElement).setView([32.7157, -117.1611], 13)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map)
  fetch('data/transit_stops_datasd.geojson')
    .then(res => res.json())
    .then(data => {
      L.geoJSON(data, {
        pointToLayer: (feature, latlng) => {
          return L.circleMarker(latlng, {
            radius: 4,                 // size of the dot
            fillColor: 'red',          // inside color
            color: 'red',              // border color
            weight: 1,                 // border thickness
            opacity: 1,                // border opacity
            fillOpacity: 0.8           // fill opacity
          })
        }
      }).addTo(map)
    })
  fetch('data/transit_routes_datasd.geojson')
    .then(res => {
      return res.json()
    })
    .then(data => {
      L.geoJSON(data).addTo(map)
    })
  fetch('data/Census Tracts 2010_20250731.geojson')
    .then(res => {
      return res.json()
    })
    .then(data => {
      L.geoJSON(data).addTo(map)
    })

})
</script>
<style scoped>
#map {
  height: 100vh;
  width: 100vw;
}
</style>