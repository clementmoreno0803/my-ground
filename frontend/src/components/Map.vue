<template>
    <div id="map" ref="mapContainer" ></div>
</template>

<script setup lang="ts">
import  mapboxgl from "mapbox-gl"
import { onMounted, ref, watch } from 'vue';
import { coordonees } from "../models/coordonees"

const position = ref<coordonees>();
    const mapContainer = ref(null);
    let map;

    const getLocation = () => {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          handlePositionSuccess,
          handlePositionError
        );
      } else {
        alert("La géolocalisation n'est pas supportée par votre navigateur.");
      }
    };

    const handlePositionSuccess = (pos: coordonees) => {
      position.value = {
        coords: {
          latitude: pos.coords.latitude,
          longitude: pos.coords.longitude,
        }
      };
      sendPositionToServer(position.value);
    };

    const handlePositionError = (error) => {
      switch (error.code) {
        case error.PERMISSION_DENIED:
          alert("Vous avez refusé l'accès à la géolocalisation.");
          break;
        case error.POSITION_UNAVAILABLE:
          alert("Les informations de localisation ne sont pas disponibles.");
          break;
        case error.TIMEOUT:
          alert("Le temps de demande a expiré.");
          break;
        default:
          alert("Une erreur inconnue est survenue.");
      }
    };

    const sendPositionToServer = (position) => {
      fetch('/api/location', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(position),
      })
      .then(response => response.json())
      .then(data => {
        console.log('Position envoyée au serveur:', data);
      })
      .catch(error => {
        console.error('Erreur lors de l\'envoi de la position:', error);
      });
};

    const initializeMap = (position: coordonees) => {
      mapboxgl.accessToken = 'pk.eyJ1IjoicGFzdGVsY2xvdGhpbmciLCJhIjoiY20xNnpuMXAzMG9nNzJrczh3ZGQ2cHRjbyJ9.sQQHL7dAIIjEuxblFzY-UQ';
      map = new mapboxgl.Map({
        container: mapContainer.value, // ID ou référence de l'élément DOM
        style: 'mapbox://styles/mapbox/streets-v11', // Style de la carte
        center: [position.coords.longitude, position.coords.latitude], // Position initiale [longitude, latitude] (Paris ici)
        zoom: 14 // Niveau de zoom initial
      });
    };

 onMounted(() => {
  getLocation()
})

watch((position), () => {
initializeMap(position.value)
})
</script>

<style scoped>
button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #38a371;
}
#map {
  width: 100vw;
  height: 100vh;
  position: absolute;
  left: 0;
  top: 0;
}
</style>
