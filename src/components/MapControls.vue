<template>
  <div class="map-controls">
    <div class="map-style-preview" @click="togglePanel">
      <div class="preview-image">
        <img :src="currentMapPreview" :alt="selectedMapStyle" />
        <div class="preview-overlay">
          <div class="preview-content">
            <CogIcon class="icon-sm" />
            <span>{{ mapStyleNames[selectedMapStyle] }}</span>
          </div>
        </div>
      </div>
    </div>
    
    <div class="control-panel" :class="{ 'panel-open': isPanelOpen }">
      <div class="panel-header">
        <h3>Map Settings</h3>
        <button class="close-button" @click="togglePanel" title="Close">
          <XMarkIcon class="icon-sm" />
        </button>
      </div>
      
      <div class="control-item">
        <div class="item-header">
          <GlobeAltIcon class="icon-sm" />
          <span>Map Style</span>
        </div>
        <div class="map-style-grid">
          <button 
            v-for="(preview, style) in mapPreviews" 
            :key="style"
            class="style-button" 
            :class="{ active: selectedMapStyle === style }"
            @click="changeMapStyle(style)"
            :title="mapStyleNames[style]">
            <div class="style-preview">
              <img :src="preview" :alt="style" />
            </div>
            <div class="style-label">
              <component :is="mapIcons[style]" class="icon-sm" />
              <span>{{ mapStyleNames[style] }}</span>
            </div>
          </button>
        </div>
      </div>
      
      <div class="control-item">
        <label class="toggle-label">
          <div class="item-header">
            <MapIcon class="icon-sm" />
            <span>Show Traffic</span>
          </div>
          <div class="toggle-switch" :class="{ 'active': showTraffic }">
            <input type="checkbox" v-model="showTraffic" @change="toggleTraffic">
            <span class="slider"></span>
          </div>
        </label>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import {
  CogIcon, XMarkIcon, MapIcon, GlobeAltIcon,
  PhotoIcon, GlobeAmericasIcon
} from '@heroicons/vue/24/outline';

const mapPreviews = {
  streets: 'https://imgur.com/GzsXaY2.jpg',
  satellite: 'https://imgur.com/0JHRPoP.jpg',
  terrain: 'https://imgur.com/VuRst8f.jpg' 
};

const mapStyleNames = {
  streets: 'Đường phố',
  satellite: 'Vệ tinh',
  terrain: 'Địa hình'
};

const mapIcons = {
  streets: MapIcon,
  satellite: PhotoIcon,
  terrain: GlobeAmericasIcon
};

const isPanelOpen = ref(false);
const selectedMapStyle = ref('streets');
const showTraffic = ref(false);

const emit = defineEmits(['changeMapStyle', 'toggleTraffic']);

const togglePanel = () => {
  isPanelOpen.value = !isPanelOpen.value;
};

const changeMapStyle = (style: string) => {
  selectedMapStyle.value = style;
  emit('changeMapStyle', style);
};

const toggleTraffic = () => {
  emit('toggleTraffic', showTraffic.value);
};

const currentMapPreview = computed(() => mapPreviews[selectedMapStyle.value]);

</script>

<style scoped>
.map-controls {
  position: fixed;
  bottom: 20px;
  left: 20px;
  z-index: 1100;
}

.control-button {
  background: rgb(255, 255, 255);
  padding: 8px;
  border-radius: 8px;
  box-shadow: var(--shadow-sm);
  cursor: pointer;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  width: 40px;
  height: 40px;
}

.control-button:hover {
  background: var(--background-color);
  box-shadow: var(--shadow-md);
  transform: translateY(-1px);
}

.control-panel {
  position: absolute;
  left: 0;
  bottom: 60px;
  background: rgb(255, 255, 255);
  padding: 20px;
  border-radius: 12px;
  box-shadow: var(--shadow-md);
  width: 280px;
  transform: translateX(-120%);
  transition: transform 0.3s ease;
  z-index: 1100;
}

.panel-open {
  transform: translateX(0);
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  z-index: 1110;
}

.close-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
  border-radius: 6px;
  color: #6b7280;
}

.close-button:hover {
  background: #f3f4f6;
}

.control-item {
  margin: 20px 0;
}

.control-label {
  display: block;
  font-size: 0.9rem;
  color: #4b5563;
  margin-bottom: 8px;
}

.select-wrapper {
  display: none;
}

select {
  display: none;
}

.select-icon {
  display: none;
}

.toggle-label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
}

.toggle-text {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #4b5563;
}

.toggle-switch {
  position: relative;
  width: 44px;
  height: 24px;
  z-index: 1120;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #e5e7eb;
  transition: .3s;
  border-radius: 24px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .3s;
  border-radius: 50%;
}

input:checked + .slider {
  background-color: #42b983;
}

input:checked + .slider:before {
  transform: translateX(20px);
}

h3 {
  font-size: 1.1rem;
  color: #111827;
  font-weight: 600;
  margin: 0;
}

/* Add these new icon styles */
.icon {
  width: 24px;
  height: 24px;
  color: var(--secondary-color);
}

.icon-sm {
  width: 20px;
  height: 20px;
  color: var(--secondary-color);
}

.icon-xs {
  width: 16px;
  height: 16px;
  color: var(--secondary-color);
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}

.item-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

/* Add these new styles */
.map-style-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin-top: 10px;
}

.style-button {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  background: white;
  cursor: pointer;
  transition: all 0.2s ease;
  overflow: hidden;
}

.style-preview {
  width: 100%;
  aspect-ratio: 16/9;
  overflow: hidden;
  border-radius: 6px 6px 0 0;
}

.style-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.style-label {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px;
  width: 100%;
  background: var(--background-color);
  font-size: 0.8rem;
}

.style-button:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-sm);
}

.style-button.active {
  border-color: var(--primary-color);
}

.style-button.active .style-label {
  background: var(--primary-color);
  color: white;
}

.style-button.active .icon-sm {
  color: white;
}

.map-style-preview {
  width: 120px;
  height: 120px;
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  box-shadow: var(--shadow-sm);
  transition: all 0.2s ease;
}

.map-style-preview:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

.preview-image {
  width: 100%;
  height: 100%;
  position: relative;
}

.preview-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.preview-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.2),
    rgba(0, 0, 0, 0.7)
  );
  display: flex;
  align-items: flex-end;
  padding: 12px;
}

.preview-content {
  color: white;
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
}

.preview-content .icon-sm {
  color: white;
}

.preview-content span {
  font-size: 0.9rem;
  font-weight: 500;
}

.preview-text {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 12px;
  font-size: 0.9rem;
  color: var(--text-color);
  flex: 1;
}

/* Update mobile styles */
@media (max-width: 768px) {
  .control-panel {
    position: fixed;
    left: 10px;
    right: 10px;
    bottom: 70px;
    width: calc(100% - 20px);
    max-width: 400px;
    margin: 0 auto;
    transform: translateY(120%);
  }

  .panel-open {
    transform: translateY(0);
  }

  .map-style-grid {
    grid-template-columns: 1fr;
    gap: 8px;
  }

  .style-button {
    flex-direction: row;
    height: 60px;
  }

  .style-preview {
    width: 60px;
    aspect-ratio: 1;
    border-radius: 6px 0 0 6px;
  }

  .style-label {
    flex: 1;
    padding: 8px 12px;
    font-size: 0.9rem;
    justify-content: flex-start;
  }

  .map-style-preview {
    width: 100px;
    height: 100px;
  }

  .preview-content span {
    font-size: 0.8rem;
  }

  .preview-text span {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}
</style>
