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
import { mapData } from './map';                          // add dummy data
import 'leaflet.markercluster';                           // add marker clustering functionality
import MapFeatures from "@/components/MapFeatures.vue"    // add component containing search

// function that creates Leaflet Icons
function createCategoryPin(fileName: string){
  return L.icon({iconUrl: require(`../assets/category-pins/${fileName}.svg`), iconSize: [50, 50]});
}

function createPhotoPin(fileName: string){
  return L.icon({iconUrl: require(`../assets/photo-pins/${fileName}.svg`), iconSize: [100, 100]});
}

// create L.Icons for map
const categoryPinResort = createCategoryPin("resort");
const categoryPinWildlife = createCategoryPin("wildlife");
const categoryPinRestaurant = createCategoryPin("restaurant");
const photoPinResort = createPhotoPin("resort");
const photoPinWildlife = createPhotoPin("wildlife");
const photoPinRestaurant = createPhotoPin("restaurant");

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

      // process dummy data to markers
      const markers = L.geoJSON(mapData, {
        onEachFeature: function (feature: any, layer) {
          const popupContent = '<h4>' + feature.properties.name + '</h4>' 
              + feature.properties.description;
          
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

      // add cluster group to map
      const markerClusterGroup = L.markerClusterGroup({
        showCoverageOnHover: false,   // set to no showing bounds of markers
        disableClusteringAtZoom: 7,   // set zoom level clustering is disabled
        iconCreateFunction: function(cluster) {
        return L.divIcon({ 
          className: "cluster-group",
          html: '<div>' + cluster.getChildCount() + '</div>' })
        }
      }).addLayer(markers);
      map.addLayer(markerClusterGroup); 
    }); // end of onMounted

    //selected location
    const currentMarkers: L.Marker<any>[] = [];
    
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
                marker.setIcon(photoPinResort);
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

    return {selectLocation}
  }
});
</script>


