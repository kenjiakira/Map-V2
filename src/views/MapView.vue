<template>
  <div class="map-container">
    <SearchControl @selectLocation="handleSearchSelect" />
    <div id="map" ref="mapRef"></div>
    <MapControls 
      @changeMapStyle="handleMapStyleChange"
      @toggleTraffic="handleTrafficToggle"
    />
    <GpsControl @locationUpdate="handleLocationUpdate" />
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, onUnmounted } from 'vue';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';
import MapControls from '../components/MapControls.vue';
import GpsControl from '../components/GpsControl.vue';
import SearchControl from '../components/SearchControl.vue';

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
const searchMarkers = ref<L.Marker[]>([]); // Thêm array để lưu search markers
const DEFAULT_CENTER = [21.0285, 105.8542];
const DEFAULT_ZOOM = 13;

const userMarker = ref<L.Marker | null>(null);
const userCircle = ref<L.Circle | null>(null);

// Thêm custom icon cho search markers
const searchIcon = L.icon({
  iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41]
});

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

const handleLocationUpdate = (location: { lat: number; lng: number; accuracy: number }) => {
  if (!map.value) return;

  const newLatLng = [location.lat, location.lng] as L.LatLngExpression;

  // Update or create user marker
  if (!userMarker.value) {
    userMarker.value = L.marker(newLatLng).addTo(map.value);
  } else {
    userMarker.value.setLatLng(newLatLng);
  }

  // Update or create accuracy circle
  if (!userCircle.value) {
    userCircle.value = L.circle(newLatLng, {
      radius: location.accuracy,
      color: '#4A90E2',
      fillColor: '#4A90E2',
      fillOpacity: 0.2,
      weight: 1
    }).addTo(map.value);
  } else {
    userCircle.value.setLatLng(newLatLng);
    userCircle.value.setRadius(location.accuracy);
  }

  // Pan map to new location
  map.value.panTo(newLatLng);
};

const handleSearchSelect = (location: { lat: number; lng: number; name: string }) => {
  if (!map.value) return;
  
  // Sử dụng icon mới cho marker
  const marker = L.marker([location.lat, location.lng], {
    icon: searchIcon
  })
    .addTo(map.value)
    .bindPopup(location.name)
    .openPopup();
    
  // Lưu marker vào array
  searchMarkers.value.push(marker);
  
  // Di chuyển map đến vị trí mới
  map.value.setView([location.lat, location.lng], 16);
};

// Thêm function để xóa markers (optional)
const clearSearchMarkers = () => {
  searchMarkers.value.forEach(marker => {
    if (map.value) {
      map.value.removeLayer(marker);
    }
  });
  searchMarkers.value = [];
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

// Optional: Cleanup khi component unmount
onUnmounted(() => {
  clearSearchMarkers();
});
</script>

<style>
.map-container {
  position: fixed;
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
