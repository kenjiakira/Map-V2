<template>
  <div class="gps-control" :class="{ active: isTracking }">
    <button class="gps-button" @click="toggleGPS" :title="isTracking ? 'Đang định vị' : 'Vị trí của tôi'">
      <div class="gps-icon" :class="{ 'gps-active': isTracking }">
        <!-- Google Maps style GPS icon -->
        <svg viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 8c-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4-1.79-4-4-4zm8.94 3c-.46-4.17-3.77-7.48-7.94-7.94V1h-2v2.06C6.83 3.52 3.52 6.83 3.06 11H1v2h2.06c.46 4.17 3.77 7.48 7.94 7.94V23h2v-2.06c4.17-.46 7.48-3.77 7.94-7.94H23v-2h-2.06zM12 19c-3.87 0-7-3.13-7-7s3.13-7 7-7 7 3.13 7 7-3.13 7-7 7z"/>
        </svg>
      </div>
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, onUnmounted } from 'vue';

const emit = defineEmits(['locationUpdate']);
const isTracking = ref(false);
let watchId: number | null = null;

const startTracking = () => {
  if ("geolocation" in navigator) {
    watchId = navigator.geolocation.watchPosition(
      (position) => {
        emit('locationUpdate', {
          lat: position.coords.latitude,
          lng: position.coords.longitude,
          accuracy: position.coords.accuracy
        });
      },
      (error) => {
        console.error('GPS Error:', error);
        isTracking.value = false;
      },
      {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0
      }
    );
    isTracking.value = true;
  }
};

const stopTracking = () => {
  if (watchId !== null) {
    navigator.geolocation.clearWatch(watchId);
    watchId = null;
  }
  isTracking.value = false;
};

const toggleGPS = () => {
  if (isTracking.value) {
    stopTracking();
  } else {
    startTracking();
  }
};

onUnmounted(() => {
  stopTracking();
});
</script>

<style scoped>
.gps-control {
  position: fixed;
  right: 20px; 
  bottom: 120px;
  z-index: 1000;
}

.gps-button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: white;
  border: none;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
  cursor: pointer;
  padding: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.gps-button:hover {
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.gps-icon {
  width: 22px;
  height: 22px;
  color: #666;
  transition: all 0.3s ease;
}

.gps-icon.gps-active {
  color: #4285f4;
}

.gps-active {
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .gps-control {
    right: 15px;
    bottom: 100px;
  }

  .gps-button {
    width: 36px;
    height: 36px;
    padding: 6px;
  }

  .gps-icon {
    width: 20px;
    height: 20px;
  }
}
</style>
