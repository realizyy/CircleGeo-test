<template>
    <div class="map-container">
        <layerControl @toggle-layer="toggleLayer" />
        <div id="map" ref="mapContainer"></div>
    </div>
</template>

<script setup>
import layerControl from './layerControl.vue';
import { ref, onMounted, onUnmounted } from 'vue';
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';
import geojsonData from '~/public/test.json';

const mapContainer = ref(null);
let map = null;
const layerVisibility = ref({
  line: true,
  polygon: true,
  circle: true,
  customMarker: true,
  defaultMarker: true,
  vectorTiles: true,
});

const initializeMap = () => {
  map = new maplibregl.Map({
    container: mapContainer.value,
    style: 'https://demotiles.maplibre.org/style.json',
    center: [118, -2],
    zoom: 4
  });

  map.on('load', () => {
    addLayers();
  });
};

const addLayers = () => {
  // line layer
  map.addLayer({
    id: 'line',
    type: 'line',
    source: {
      type: 'geojson',
      data: geojsonData
    },
    paint: {
      'line-color': ['get', 'lineColor'],
      'line-width': 2
    },
    filter: ['==', '$type', 'LineString'],
    paint: {
        "line-color": [
            "case",
            ["boolean", ["feature-state", "hover"], false],
            "#ff0000",
            "#ff0000"
        ],
        "line-width": 2
    },
  });

  // Polygon layer
  map.addLayer({
    id: 'polygon',
    type: 'fill',
    source: {
      type: 'geojson',
      data: geojsonData
    },
    paint: {
      'fill-color': ['get', 'fillColor'],
      'fill-opacity': 0.5
    },
    filter: ['==', '$type', 'Polygon']
  });

  // Circle layer
  map.addLayer({
    id: 'circle',
    type: 'circle',
    source: {
      type: 'geojson',
      data: geojsonData
    },
    paint: {
      'circle-radius': ['get', 'radius'],
      'circle-color': '#007cbf'
    },
    filter: ['all', ['==', '$type', 'Point'], ['==', 'name', 'Circle Marker']],
  });

  // Custom Image
  const images = [
    'https://circlegeo.com/wp-content/uploads/2023/08/logo-cg-150x150.png',
    'https://circlegeo.com/wp-content/uploads/2023/08/logo-cg-150x150.png'
  ];

  // Custom marker layer
  geojsonData.features
    .filter(feature => feature.properties.name === 'Custom Image Marker')
    .forEach(feature => {
      const el = document.createElement('div');
      el.className = 'custom-marker';
      el.style.backgroundImage = `url(${images.pop()})`;
      el.style.width = '30px';
      el.style.height = '35px';
      el.style.backgroundSize = 'cover';
      el.style.cursor = 'pointer';

      new maplibregl.Marker({ element: el })
        .setLngLat(feature.geometry.coordinates)
        .setPopup(new maplibregl.Popup().setHTML(`<h3>${feature.properties.name}</h3>`))
        .addTo(map);
    });

  // Default marker layer
  geojsonData.features
    .filter(feature => feature.properties.name === 'Marker Default')
    .forEach(feature => {
      new maplibregl.Marker()
        .setLngLat(feature.geometry.coordinates)
        .setPopup(new maplibregl.Popup().setHTML(`<h3>${feature.properties.name}</h3>`))
        .addTo(map);
    });

  // Vector tiles layer
  map.addLayer({
    id: 'vectorTiles',
    type: 'raster',
    source: {
      type: 'raster',
      tiles: [
        'https://rami.bmkg.go.id/api/windtemp_get/wafc/WT/snow/850/202405121800/202405131200/{z}/{x}/{y}.png'
      ],
      tileSize: 256
    },
    paint: {}
  });

  // Popup
  map.on('click', (e) => {
    const features = map.queryRenderedFeatures(e.point, {
      layers: ['line', 'polygon', 'circle']
    });

    if (!features.length) return;

    const feature = features[0];
    new maplibregl.Popup()
      .setLngLat(e.lngLat)
      .setHTML(`<h3>${feature.properties.name || 'Feature'}</h3>`)
      .addTo(map);
  });
};

const toggleLayer = (layerId) => {
  layerVisibility.value[layerId] = !layerVisibility.value[layerId];
  const visibility = layerVisibility.value[layerId] ? 'visible' : 'none';
  
  if (layerId === 'customMarker' || layerId === 'defaultMarker') {
    const markers = document.querySelectorAll(`.${layerId === 'customMarker' ? 'custom-marker' : 'maplibregl-marker'}`);
    markers.forEach(marker => {
      marker.style.display = visibility === 'visible' ? 'block' : 'none';
    });
  } else {
    map.setLayoutProperty(layerId, 'visibility', visibility);
  }
};

onMounted(() => {
  initializeMap();
});

onUnmounted(() => {
  if (map) map.remove();
});
</script>

<style scoped>
.map-container {
  position: relative;
  height: 100vh;
  width: 100%;
}

#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}

.custom-marker {
  background-size: cover;
}

.default-marker {
  background-size: cover;
}
</style>