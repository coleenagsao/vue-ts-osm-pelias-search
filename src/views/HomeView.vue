<template>
  <div class="home">
    <div id=map></div>
    <div class="features">
      <MapFeatures @selectLocation="selectLocation"/>
    </div>
  </div>
</template>

<style>
  .home{
    height: 100vh;
    position: relative;
  }

  .features, #map {
    position: absolute;
    top: 0;
    left: 0;
  }

  .features {
    background-color: rgba(0, 0, 0, 0);
    z-index: 2;
    width: 100%;
    height: 0%;
  }

  #map {
    z-index: 1;
    width: 100%;
    height: 100%;
  }

  .cluster-group div {
    width: 30px;
    height: 30px;
    border-radius: 50%;
    background-color: whitesmoke;
    box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5);

    display: flex;
    text-align: center;
    justify-content: center;
    align-items: center;

    font-size: 12px;
  }

</style>

<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue';
import L, { LatLng } from "leaflet";
import MapFeatures from "@/components/MapFeatures.vue"

import { mapData } from './map';
import 'leaflet.markercluster';

//category pins
function createCategoryPin(fileName: string){
  return L.icon({iconUrl: require(`../assets/category-pins/${fileName}.svg`), iconSize: [50, 50]});
}

const categoryPinResort = createCategoryPin("resort");
const categoryPinWildlife = createCategoryPin("wildlife");
const categoryPinRestaurant = createCategoryPin("restaurant");


export default defineComponent({
  name: 'HomeView',
  components: {MapFeatures},
  setup(){
    let map: L.Map;

    onMounted(() => {
      //initialize map with default top-left zoom control off
      map = L.map('map', {zoomControl: false}).setView([13.41, 122.56], 5);

      // add tile layer
      L.tileLayer(
       "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
       {
        maxZoom: 18,
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
       }
     ).addTo(map);

     // position zoom control in bottom left
     L.control.zoom({ position: 'bottomleft' }).addTo(map);
    
     // get user's current coordinate
     if (sessionStorage.getItem("coords")){
        UserCoords.value = JSON.parse(sessionStorage.getItem('coords') || 'null'); 
        
        const marker = L.geoJSON(mapData, {
          onEachFeature: function (feature: any, layer) {
                const popupContent =
                '<h4>' + feature.properties.name +  '</h4>' + feature.properties.description;
                layer.bindTooltip(popupContent);

          },
          pointToLayer: function (feature, latlng) {
            if (feature.properties.type == "Adventure"){
              return new L.Marker(latlng, {icon: categoryPinWildlife});
            } else if (feature.properties.type == "Food & Drink") {
              return new L.Marker(latlng, {icon: categoryPinRestaurant});
            } else {
              return new L.Marker(latlng, {icon: categoryPinResort}); //temp
            }     
          },
        });

        const markers = L.markerClusterGroup({
            //unable showing bounds of markers
            showCoverageOnHover: false,
            disableClusteringAtZoom: 7, // set zoom level clustering is disabled
            iconCreateFunction: function(cluster) {
              return L.divIcon({ 
                className: "cluster-group",
                html: '<div>' + cluster.getChildCount() + '</div>' })
            }
        }).addLayer(marker);

        // marker clustering
        map.addLayer(markers); 
  
        return;
      }

      fetchCoords.value = true;
      navigator.geolocation.getCurrentPosition(setUserCoords, getUserLocError);

    });
  
    // icons




    const photoMarker = L.icon({
      iconUrl: require("../assets/photo-pins/resort.svg"),
      iconSize: [100, 100]
    })

    // user's current location
    const UserCoords = ref<any>(null);
    const fetchCoords = ref(false);
    const UserCoordsError = ref<boolean | null>(null);
    const UserCoordsErrorMsg = ref(null); 

    //selected location
    const currentMarkers: L.Marker<any>[] = [];

    const setUserCoords = (pos: any) => {
      // stop fetching coords
      fetchCoords.value = false;

      // set user's coords in session storage
      const SessionCoords = {
        lat: pos.coords.latitude as number,
        lng: pos.coords.longitude as number
      };

      //add to session storage
      sessionStorage.setItem('coords', JSON.stringify(SessionCoords));

      // set ref coords value
      UserCoords.value = SessionCoords;
      //plotPin(UserCoords.value);
    }

    const getUserLocError =  (err: any) => {
      fetchCoords.value = false;
      UserCoordsError.value = true;
      UserCoordsErrorMsg.value = err.message;
    }

    const closeUserCoordsError = () => {
      UserCoordsError.value = null;
      UserCoordsErrorMsg.value = null;
    }

    const plotPin = (coords: { lat: number, lng: number }) => {
      for (const marker of currentMarkers) {
        map.removeLayer(marker);
    }

      if (coords !== null){
        console.log(coords.lat);
        console.log(coords.lng);
        const position: LatLng = new L.LatLng(coords.lat, coords.lng);
        currentMarkers.push(new L.Marker(position, {icon: categoryPinRestaurant}).addTo(map));

        map.flyTo(position, 12);

        for (const marker of currentMarkers) {
          map.on('zoomend', function() {
            //console.log(map.getZoom());
              if (map.getZoom() > 7) {
                marker.setIcon(photoMarker);
              } else {
                marker.setIcon(categoryPinRestaurant);
              }
          });
        }
      }
    };

    const selectLocation = (coords: { lat: number, lng: number }) => {
      if (coords !== null){
        console.log(coords.lat);
        console.log(coords.lng);

        const position: LatLng = new L.LatLng(coords.lat, coords.lng);
        map.flyTo(position, 12);
      }
    };

    return {UserCoords, closeUserCoordsError, selectLocation}
  }
});
</script>


