<template>
  <div class="main-content">
    <h1 class="display-4 mb-3">San Diego Transit Map</h1>
    <h2><i>Transit in America's Finest City</i></h2>

    <p>Transit is important in cities. In San Diego there is the trolley and bus
      system. This map shows the transit routes and stops in San Diego, as well as
      the number of transit stops per census tract. The census tracts are colored
      by the number of transit stops they contain.
    </p>

    My map ideas are:
    <ul>
      <li>To create an intro map, showing which areas have the most transit stops and which don't</li>
      <li>To create a ranked score based on proximity to stop, grocery, poverty levels, and car ownership</li>
      <li>How long is your commute to work map?</li>
    </ul>

    <div id="map1" class="map">
      <div class="map-overlay" id="legend"></div>
    </div>

    <h2>Transit Challenges</h2>
    <p>Despite San Diego's beautiful weather, our transit system has challenges.
      Many communities lack direct access to a nearby stop. Equity and access
      remain ongoing concerns.
    </p>

    <p>The map above shows the number of transit stops in each census tract.
      Areas with fewer stops may face greater challenges in accessing public transit.
    </p>

    <p>The trolley and bus routes are not as extensive as they could be, especially
      in suburban areas. This can lead to longer commutes and less reliable service.
    </p>

    <p>Improving transit access is crucial for reducing traffic congestion and
      promoting sustainable transportation options.
    </p>
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
    center: [-117.11056697395156, 32.97857887589738],
    zoom: 8.0
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

    map1.addLayer({
      // change to number of stops
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
        'fill-opacity': 0.9,
        // Optional: Add outline when hovered
        'fill-outline-color': '#000'
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
