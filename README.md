# OSM with Pelias Search

## Description
This repo focuses on the features of OpenStreetMap (OSM) and Pelias. It utilizes OpenStreetMap (OSM) for the map functionality. It also integrates the self-hosted Pelias geocoding service to provide a user-friendly search bar with autocomplete endpoint enabled. This allows users to interact with basic map functionalities like searching for locations and zoom in to pre-determined added to map using geojson data. Additionally, once a search result is selected, users can easily navigate to the desired places. 

## Technologies Used
![Vue.js](https://img.shields.io/badge/Vue.js-42b883?style=for-the-badge&logo=vue.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-2d79c7?style=for-the-badge&logo=typescript&logoColor=white)

## Installation
To run the project locally, please follow these steps:

1. Begin by cloning the repository to your local machine using the command: `https://github.com/coleenagsao/vue-ts-pelias-data-ingestion-poc.git`
2. Install the necessary dependencies by executing: `npm install`
3. Begin the website by running the command: `npm run serve`.
4. Ensure that your self-hosted Pelias instance is up and running on your local machine when searching for places. 

Before proceeding with the installation, make sure you have the necessary prerequisites installed on your machine, including self-hosted Pelias, Vue, and TypeScript (TS). You may also opt to change the OSM url to your own locally hosted OSM.

You can refer to the official Pelias Docker repository by clicking [here](https://github.com/pelias/docker). This repository provides detailed instructions and guidance on setting up and running Pelias locally on your machine.