<template>
  <div class="flex-col ">
    

  <div class="h-screen sticky bg-slate-600" >
    <GeoError @closeGeoError="closeGeoError" v-if="geoError" :geoErrorMsg="geoErrorMsg"/>
    <MapFeatures @getGeoLocation="getGeoLocation" @plotResult="plotResult" :coords="coords" :fetchCoords="fetchCoords" :searchResults="searchResults" @toggleSearchResults="toggleSearchResults" @removeResult="removeResult"  />
    <div id="map" class="h-5/6 my-30 z-[1]"></div>
  </div>
  <div class="bg-slate-600 flex justify-center content-start">
  <img class="bg-slate-600" src="../assets/gill.png" alt="gill-testimonial">
  </div>
</div>
</template>

<script>
import leaflet from 'leaflet';
import GeoError from '../components/GeoError.vue';
import { onMounted, ref } from 'vue';
import MapFeatures from '@/components/MapFeatures.vue';

export default {
  name: 'HomeView',
  components: {
    GeoError,
    MapFeatures
  } ,
  setup() {
    let map;
    onMounted(() => {
      map = leaflet.map('map').setView([40.4406, -79.9959], 13);

      leaflet.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
}).addTo(map);

  map.on('moveend', () => {
    closeSearchResults();
  })
  getGeoLocation();
})
   
  const coords = ref(null);
  const fetchCoords = ref(null);
  const geoMarker = ref(null);
  const geoError = ref(null);
  const geoErrorMsg = ref(null);

  const getGeoLocation = () => {

    if(coords.value) {
      coords.value = null;
      sessionStorage.removeItem("coords");
      map.removeLayer(geoMarker.value);
      return;
    }

    //Check for coords in storage
    if(sessionStorage.getItem('coords')) {
      coords.value = JSON.parse(sessionStorage.getItem("coords"));
      plotGeoLocation(coords.value);
      return;
    }

    fetchCoords.value = true;
    navigator.geolocation.getCurrentPosition(setCoords, getLocError);
  };

  const setCoords = (pos) => {
      fetchCoords.value = null;

      const setSessionCoords = {
        lat: pos.coords.latitude,
        lng: pos.coords.longitude
      };

      sessionStorage.setItem('coords', JSON.stringify(setSessionCoords));

      coords.value = setSessionCoords;

      plotGeoLocation(coords.value);
  };

  const getLocError = (err) => {
    fetchCoords.value = null;

    geoError.value = true;
    geoErrorMsg.value = err.message;
  };

  const plotGeoLocation = (coords) => {
    const customMarker = leaflet.icon({iconUrl: require('../assets/redMarker.png'),
    iconSize: [35, 35],
  });

    geoMarker.value = leaflet.marker([coords.lat, coords.lng, {icon: customMarker}]).addTo(map);


    map.setView([coords.lat, coords.lng], 10);

  };

  const closeGeoError = () => {
    geoError.value = null;
    geoErrorMsg.value = null;
  };



  const resultMarker = ref(null);
  const plotResult = (coords) => {
      if (resultMarker.value) {
        map.removeLayer(resultMarker.value);
      }

      const customMarker = leaflet.icon({iconUrl: require('../assets/map-marker-gold.svg'),
    iconSize: [35, 35],
  });

    resultMarker.value = leaflet.marker([coords.coordinates[1], coords.coordinates[0], {icon: customMarker}]).addTo(map);


    map.setView([coords.coordinates[1], coords.coordinates[0]], 14);

    closeSearchResults();
  };

  const searchResults = ref(null);
  const toggleSearchResults = () => {
    searchResults.value = !searchResults.value;
  };

  const closeSearchResults = () => {
    searchResults.value = null;
  };

  const removeResult = () => {
    map.removeLayer(resultMarker.value);
  }

  return { coords, fetchCoords, geoMarker, closeGeoError, geoError, geoErrorMsg, getGeoLocation, plotResult, searchResults, toggleSearchResults, closeSearchResults, removeResult};
  
  },
 
};
</script>
