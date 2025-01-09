<template>
  <div class="search-container">
    <div class="search-box">
      <div class="search-input-wrapper">
        <MagnifyingGlassIcon class="search-icon" />
        <input
          type="text"
          v-model="searchQuery"
          @input="handleInput"
          @focus="showHistory"
          placeholder="Tìm kiếm địa điểm..."
          ref="searchInput"
        />
        <button v-if="searchQuery" @click="clearSearch" class="clear-button">
          <XMarkIcon class="clear-icon" />
        </button>
      </div>
      
      <div class="search-results" v-if="isSearching && (results.length > 0 || searchHistory.length > 0)">
        <!-- Lịch sử tìm kiếm -->
        <div v-if="!searchQuery && searchHistory.length > 0" class="history-section">
          <div class="history-header">
            <ClockIcon class="icon-sm" />
            <span>Tìm kiếm gần đây</span>
            <button @click="clearHistory" class="clear-history">Xóa</button>
          </div>
          <div
            v-for="item in searchHistory"
            :key="item.id"
            class="search-result-item"
            @click="handleHistorySelect(item)"
          >
            <ClockIcon class="location-icon" />
            <div class="result-content">
              <div class="result-name">{{ item.name }}</div>
              <div class="result-address">{{ item.address }}</div>
            </div>
          </div>
        </div>

        <!-- Kết quả tìm kiếm -->
        <div v-if="results.length > 0" class="results-section">
          <div
            v-for="result in results"
            :key="result.place_id"
            class="search-result-item"
            @click="handleSelectLocation(result)"
          >
            <MapPinIcon class="location-icon" />
            <div class="result-content">
              <div class="result-name">{{ result.display_name.split(',')[0] }}</div>
              <div class="result-address">{{ result.display_name }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { MagnifyingGlassIcon, XMarkIcon, MapPinIcon, ClockIcon } from '@heroicons/vue/24/outline';

const emit = defineEmits(['selectLocation']);
const searchQuery = ref('');
const isSearching = ref(false);
const results = ref<any[]>([]);
const searchInput = ref<HTMLInputElement | null>(null);
let debounceTimer: number | null = null;

const MAX_HISTORY = 5;
const DEBOUNCE_DELAY = 200;
const searchHistory = ref<any[]>([]);

const loadHistory = () => {
  const saved = localStorage.getItem('searchHistory');
  if (saved) {
    searchHistory.value = JSON.parse(saved);
  }
};

const saveHistory = (item: any) => {
  const newItem = {
    id: Date.now(),
    name: item.display_name.split(',')[0],
    address: item.display_name,
    lat: item.lat,
    lng: item.lon
  };
  
  searchHistory.value = [
    newItem,
    ...searchHistory.value.filter(h => h.address !== newItem.address)
  ].slice(0, MAX_HISTORY);
  
  localStorage.setItem('searchHistory', JSON.stringify(searchHistory.value));
};

const clearHistory = () => {
  searchHistory.value = [];
  localStorage.removeItem('searchHistory');
};

const showHistory = () => {
  if (!searchQuery.value) {
    isSearching.value = true;
  }
};

const handleHistorySelect = (item: any) => {
  emit('selectLocation', {
    lat: parseFloat(item.lat),
    lng: parseFloat(item.lng),
    name: item.address
  });
  closeSearch();
};

const handleSelectLocation = (result: any) => {
  saveHistory(result);
  emit('selectLocation', {
    lat: parseFloat(result.lat),
    lng: parseFloat(result.lon),
    name: result.display_name
  });
  closeSearch();
};

const handleInput = () => {
  if (debounceTimer) clearTimeout(debounceTimer);
  debounceTimer = setTimeout(async () => {
    if (searchQuery.value.length < 2) {
      results.value = [];
      return;
    }
    try {
      const response = await fetch(
        `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(searchQuery.value)}&limit=5&countrycodes=vn&addressdetails=1`
      );
      results.value = await response.json();
    } catch (error) {
      console.error('Search error:', error);
    }
  }, DEBOUNCE_DELAY);
};

const clearSearch = () => {
  searchQuery.value = '';
  results.value = [];
};

const closeSearch = () => {
  isSearching.value = false;
  results.value = [];
  searchQuery.value = '';
};

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') {
    closeSearch();
  }
};

const handleClickOutside = (event: MouseEvent) => {
  const searchBox = document.querySelector('.search-box');
  if (searchBox && !searchBox.contains(event.target as Node)) {
    closeSearch();
  }
};

onMounted(() => {
  loadHistory();
  document.addEventListener('keydown', handleKeydown);
  document.addEventListener('click', handleClickOutside);
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown);
  document.removeEventListener('click', handleClickOutside);
});
</script>

<style scoped>
.search-container {
  position: fixed;
  top: 15px;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 500px;
  padding: 0 15px;
  z-index: 1200;
}

.search-box {
  position: relative;
  width: 100%;
}

.search-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  background: white;
  border-radius: 50px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.15);
}

.search-icon {
  width: 18px;
  height: 18px;
  color: #666;
  margin-left: 16px;
  flex-shrink: 0;
}

input {
  width: 100%;
  padding: 12px 16px;
  border: none;
  background: none;
  font-size: 0.95rem;
  outline: none;
  color: #333;
}

input::placeholder {
  color: #999;
}

.clear-button {
  padding: 8px 16px;
  background: none;
  border: none;
  cursor: pointer;
  color: #666;
  display: flex;
  align-items: center;
}

.clear-icon {
  width: 16px;
  height: 16px;
}

.search-results {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  right: 0;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  max-height: 380px;
  overflow-y: auto;
  z-index: 1200;
}

.search-result-item {
  display: flex;
  align-items: flex-start;
  padding: 12px 16px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.search-result-item:hover {
  background: #f8f9fa;
}

.location-icon {
  width: 16px;
  height: 16px;
  color: #666;
  margin-right: 12px;
  margin-top: 3px;
  flex-shrink: 0;
}

.result-content {
  flex: 1;
  min-width: 0;
  padding-right: 8px;
}

.result-name {
  font-weight: 500;
  color: #333;
  margin-bottom: 3px;
  font-size: 0.95rem;
}

.result-address {
  font-size: 0.8rem;
  color: #666;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Xóa phần overlay styles */
.overlay {
  display: none;
}

/* Custom Scrollbar */
.search-results::-webkit-scrollbar {
  width: 8px;
}

.search-results::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.search-results::-webkit-scrollbar-thumb {
  background: #ddd;
  border-radius: 4px;
}

.history-section {
  border-bottom: 1px solid #eee;
  padding-bottom: 8px;
}

.history-header {
  display: flex;
  align-items: center;
  padding: 12px 16px 8px;
  color: #666;
  font-size: 0.9rem;
  gap: 8px;
}

.clear-history {
  margin-left: auto;
  padding: 4px 8px;
  font-size: 0.8rem;
  color: #666;
  background: none;
  border: none;
  cursor: pointer;
}

.clear-history:hover {
  text-decoration: underline;
}

.icon-sm {
  width: 16px;
  height: 16px;
}

@media (max-width: 768px) {
  .search-container {
    top: 10px;
    padding: 0 10px;
    max-width: calc(100% - 20px);
  }

  .search-input-wrapper {
    border-radius: 25px;
  }

  input {
    padding: 10px 12px;
    font-size: 0.9rem;
  }

  .search-results {
    max-height: 60vh;
    border-radius: 10px;
  }

  .result-name {
    font-size: 0.9rem;
  }

  .result-address {
    font-size: 0.75rem;
  }
}
</style>
