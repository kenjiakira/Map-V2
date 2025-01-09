<template>
  <div class="map-controls">
    <div class="map-style-preview" @click="togglePanel">
      <div class="preview-image">
        <img :src="currentMapPreview" :alt="selectedMapStyle" />
        <div class="preview-overlay">
          <div class="preview-content">
            <span>Lớp bản đồ</span>
          </div>
        </div>
      </div>
    </div>
    
    <div class="control-panel" :class="{ 'panel-open': isPanelOpen }">
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
            <span>{{ mapStyleNames[style] }}</span>
          </div>
        </button>
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

 <style>
@import "../styles/controls.css";
</style>