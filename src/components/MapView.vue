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
      <div class="map-overlay2" id="title">Total estimate of workers (16 years and over) who have a travel time to work of 60 or more minutes</div>
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
    const [tractsGeojson, routesGeojson] = await Promise.all([
      fetch('data/ACS_2023_c.geojson').then(res => res.json()),
      fetch('data/transit_routes_datasd.geojson').then(res => res.json())
    ]);
    map1.addSource('tracts', {
      type: 'geojson',
      data: tractsGeojson
    })
    map1.addSource('transit_routes', {
      type: 'geojson',
      data: routesGeojson
    })
    map1.addLayer({
      // change to number of stops
      id: 'tracts-fill',
      type: 'fill',
      source: 'tracts',
      paint: {
        // Total Estimate of workers 16 years and over who have a travel time to work of 60 or more minutes
        'fill-color': [
          'interpolate',
          ['linear'],
          ['get', 'S0802_C01_089E'], // Property to style by
          5, '#ffffcc',  // Lower values
          10, '#a1dab4',  // Middle values
          20, '#41b6c4',   // Higher values
          32, '#2c7fb8'  // Highest values
        ],
        'fill-opacity': 0.9,
        // Optional: Add outline when hovered
        'fill-outline-color': '#000'
      }
    })
    map1.addLayer({
      id: 'trasit-routes',
      type: 'line',
      source: 'transit_routes',
      paint: {
        'line-color': '#ff0000',
        'line-width': 0.3
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
    
    const layers = ['0-5', '5-10', '10-20', '20-32'];
    const colors = ['#ffffcc', '#a1dab4', '#41b6c4', '#2c7fb8'];
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
    // append transit routes to legend
    const transitItem = document.createElement('div');
    const transitKey = document.createElement('span');
    transitKey.className = 'legend-key-line';
    transitKey.style.backgroundColor = '#ff0000';
    const transitValue = document.createElement('span');
    transitValue.innerHTML = 'Transit Routes';
    transitItem.appendChild(transitKey);
    transitItem.appendChild(transitValue);
    legend.appendChild(transitItem);
  })
})
</script>
