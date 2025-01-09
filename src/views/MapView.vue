<template>
  <div class="map-container">
    <div id="map" ref="mapRef"></div>
    <MapControls 
      @changeMapStyle="handleMapStyleChange"
      @toggleTraffic="handleTrafficToggle"
    />
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';
import MapControls from '../components/MapControls.vue';

// Fix Leaflet icon issue
import iconRetinaUrl from 'leaflet/dist/images/marker-icon-2x.png';
import iconUrl from 'leaflet/dist/images/marker-icon.png';
import shadowUrl from 'leaflet/dist/images/marker-shadow.png';

delete (L.Icon.Default.prototype as any)._getIconUrl;
L.Icon.Default.mergeOptions({
  iconRetinaUrl,
  iconUrl,
  shadowUrl,
});

const mapRef = ref<HTMLElement | null>(null);
const map = ref<L.Map | null>(null);
const DEFAULT_CENTER = [21.0285, 105.8542];

const handleMapStyleChange = (style: string) => {
  if (!map.value) return;
  
  const styles = {
    streets: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
    satellite: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
    terrain: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png'
  };

  map.value.eachLayer((layer) => {
    if (layer instanceof L.TileLayer) {
      map.value?.removeLayer(layer);
    }
  });

  L.tileLayer(styles[style as keyof typeof styles], {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map.value);
};

const handleTrafficToggle = (show: boolean) => {
  if (!map.value) return;

  console.log('Traffic toggle:', show);
};

onMounted(() => {
  if (mapRef.value) {
    map.value = L.map(mapRef.value, {
      zoomControl: false,
      scrollWheelZoom: true,  
      touchZoom: true,  
      doubleClickZoom: true,
      dragging: true,    
    }).setView(DEFAULT_CENTER, DEFAULT_ZOOM);
    
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors'
    }).addTo(map.value);
  }
});
</script>

<style>
.map-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
}

#map {
  width: 100%;
  height: 100%;
}

:deep(.leaflet-pane) { 
  z-index: 400; 
}

:deep(.leaflet-control) { 
  z-index: 800; 
}

:deep(.leaflet-top), 
:deep(.leaflet-bottom) { 
  z-index: 900; 
}
</style>
