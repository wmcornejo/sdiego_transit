<template>
  <h1>San Diego Transit Map</h1>
  <p>Hover over a tract to see the number of transit stops within it.</p>
  <p>Data Source: <a href="https://data.sandiego.gov/datasets/?">San Diego Datasets</a></p>
  <div class="map-row">
    <div id="map1" class="map">
      <div class="map-overlay" id="legend">

        <strong>Legend</strong>
      </div>
    </div>
  </div>
</template>

<script setup>
import mapboxgl from 'mapbox-gl'
import { onMounted } from 'vue'
import * as turf from "@turf/turf";

// Your token
mapboxgl.accessToken = 'pk.eyJ1Ijoid202MjUiLCJhIjoiY21kc3RwZ3ZjMHdkMzJqcG9vbmRrcWxvdCJ9.QDO6nEgb2Ht2F3iAGtG_xw'

onMounted(() => {
  const map1 = new mapboxgl.Map({
    container: 'map1',
    style: 'mapbox://styles/mapbox/light-v11',
    center: [-117.11056697395156, 32.87827887583728],
    zoom: 8.5
  })
  map1.addControl(new mapboxgl.FullscreenControl());
  map1.on('load', async () => {
    const [tractsGeojson, stopsGeojson, routesGeojson] = await Promise.all([
      fetch('data/ACS_2023_b.geojson').then(res => res.json()),
      fetch('data/transit_stops_datasd.geojson').then(res => res.json()),
      fetch('data/transit_routes_datasd.geojson').then(res => res.json())
    ]);
    tractsGeojson.features.forEach(tract => {
      const ptsWithin = turf.pointsWithinPolygon(stopsGeojson, tract);
      tract.properties.transit_stop_count = ptsWithin.features.length;
    });
    map1.addSource('tracts', {
      type: 'geojson',
      data: tractsGeojson
    })
    map1.addSource('transit_routes', {
      type: 'geojson',
      data: routesGeojson
    });
    map1.addSource('transit_stops', {
      type: 'geojson',
      data: stopsGeojson
    });
    map1.addLayer({
      id: 'tracts-fill',
      type: 'fill',
      source: 'tracts',
      paint: {
        // Example: Color by median income (replace 'median_income' with your column name)
        'fill-color': [
          'interpolate',
          ['linear'],
          ['get', 'S1701_C01_001E'], // Property to style by
          0, '#ffffcc',  // Lower values
          2288, '#a1dab4',  // Middle values
          3585, '#41b6c4',   // Higher values
          4560, '#2c7fb8',  // Highest values
          5590, '#253494'
        ],
        'fill-opacity': 0.7,
        // Optional: Add outline when hovered
        'fill-outline-color': '#000'
      }
    })

    map1.addLayer({
      id: 'tracts-outline',
      type: 'line',
      source: 'tracts',
      paint: {
        'line-color': '#000',
        'line-width': 0.5
      }
    })

    map1.addLayer({
      id: 'routes',
      type: 'line',
      source: 'transit_routes',
      paint: {
        'line-color': '#0080ff',
        'line-width': 2,
        'line-opacity': 0.7
      }
    })


    map1.on('mousemove', (event) => {
      const states = map1.queryRenderedFeatures(event.point, {
        layers: ['tracts']
      });
    });

    const popup = new mapboxgl.Popup({
        closeButton: false,
        closeOnClick: false
    });
    
    const layers = ['< 120', '< 260', '< 340', '< 1000', '> 1000'];
    const colors = ['#ffffcc', '#a1dab4', '#41b6c4', '#2c7fb8', '#253494'];
    const legend = document.getElementById('legend');

    layers.forEach((layer, i) => {
      const color = colors[i];
      const item = document.createElement('div');
      const key = document.createElement('span');
      key.className = 'legend-key';
      key.style.backgroundColor = color;

      const value = document.createElement('span');
      value.innerHTML = `${layer}`;
      item.appendChild(key);
      item.appendChild(value);
      legend.appendChild(item);
    });
  })
})
</script>

