<template>
<div class="features-container">
    <div id="searchBar">
      <img src="../assets/search.png" alt="search icon">
      <input id="searchValue" v-model="query" type="text" placeholder="Negros Occidental">
      <div id="filter">
        <img src="../assets/filter.png" alt="search icon">
      </div>
      <div id="search-results-container" v-if="query && searchData">
        <div id="search-results" v-for="(result, index) in searchData" :key="index" @click="select(result)">
            <img src="../assets/marker.png" alt="search icon">
            <p>{{ result.properties.label }}</p>
        </div>
      </div>
    </div>
</div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

export default defineComponent({
  props: [],
  component: {},
  setup(props, {emit}){
    const query = ref("");
    const searchData = ref("");

    // function that gets search results from pelias (v-on:input="search")
    const search = () => {  
        fetch("http://localhost:4000/v1/autocomplete?&text=" + query.value)
            .then(result => result.json())
            .then(parsedResult => {
                console.log(parsedResult);
                searchData.value = "";

                if (query.value !== ""){
                    searchData.value = parsedResult.features;
                }        
        });
    }

    const select = (result: any) => {
        console.log(result);

        const position = { lat: result.geometry.coordinates[1], lng: result.geometry.coordinates[0] }
        emit("selectLocation", position);
    }

    return {query, select, search, searchData};
  }
})

</script>
<style>
.features-container{
    width: 100%;
    position: relative;
    display: flex;
    background-color: transparent;
    padding: 10px;
}

#searchBar {
    display: flex;
    position: relative;
    justify-content: space-between;
    align-items: center;
    width: 250px;
    height: 30px;
    margin: 10px;
    background-color: #ffffff;
    border-radius: 2px;
    padding: 5px;    
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2); 
}

#searchBar img{
    margin: 5px;
    width: 15px;
    height: 15px;
}

#searchValue {
    flex-grow: 1;
    border: none;
    background-color: transparent;
    font-size: 14px;
    outline: none;
    font-weight: bold;
}

#search-results-container {
    position: absolute;
    margin-top: 310px;
    margin-left: -5px;
    width: 100%;
    height: 250px;
    overflow: auto;
    background-color:white;
    border-radius: 2px;
}

#search-results {
    padding: 2px 5px 2px 10px;
    text-align: left; 
    font-size: 14px;
    font: #8d8fa8;
    display: flex;
    align-items: center;
}

#search-results:hover {
  background-color: rgb(240, 240, 240);
}

#search-results p{
    font-size: 12px;
    padding-left: 2px;
}

search-results img {
    width: 12px;
    height: 12px;
    padding: 15px;
}

</style>