<script setup>
import { ref } from 'vue';
import jsonDataCapitals from '@/assets/capitals.geo.json';
import jsonDataCountries from '@/assets/countries.geo.json';
import { GoogleMap, AdvancedMarker, InfoWindow, Polygon } from 'vue3-google-map';

const center = { lat: 40.689247, lng: -74.044502 };
const pinOptions = { background: '#f9350c' };

const bounds = {
	north: 85,
	south: -85,
	west: -180,
	east: 180,
};

const mapInstance = ref(null);

function getBoundBox(Polygon) {
	const bounds = new google.maps.LatLngBounds();
	const path = Polygon.latLngs.Eg.reduce((e1, e2) => (e1.length > e2.length ? e1 : e2));
	path.forEach(function(element) {
		bounds.extend(element)
	})
	return bounds;
}

function handleMarkerClick(marker) {
	if(mapInstance.value){
		mapInstance.value.map.setCenter({ lat: marker.lat, lng: marker.lng });
		mapInstance.value.map.setZoom(8);
		mapInstance.value.map.setOptions({
			restriction: {
				latLngBounds: bounds,
				strictBounds: true,
			},
		});
	}
}

function handleZoneClick(countryPolygon) {
	if(mapInstance.value){
		mapInstance.value.map.fitBounds(getBoundBox(countryPolygon))
	}
}


const countries = jsonDataCountries.features
const countryCoords = countries.map((e) => {
	if(e.geometry.type=="Polygon"){
		let polygon = e.geometry.coordinates[0].map((e2) => {
			return  { lat: e2[1], lng: e2[0]}
		})
		return polygon
	} else {
		if(e.id=="ATA") return;
		let polygons = []
		let coords = e.geometry.coordinates
		coords.forEach(subPolygon => {
			polygons.push(subPolygon[0].map((e3) => {
				return  { lat: e3[1], lng: e3[0]}
			}))
		});
		return polygons
	}
})

const countryPolygons = countryCoords.map((coords) => {
	return new google.maps.Polygon({
		paths: coords,
		strokeColor: '#FF0000',
		strokeOpacity: 0.8,
		strokeWeight: 2,
		fillColor: '#FF0000',
		fillOpacity: 0.35,
	})
})

const capitals = jsonDataCapitals.features
const markers = capitals.map((e) => {
	return  { lat: e.geometry.coordinates[1], lng: e.geometry.coordinates[0], title: e.properties.city + ', ' + e.properties.country}
})
</script>

<template>
	<GoogleMap
		ref="mapInstance"
		map-id="map"
		api-key="AIzaSyDx3EjbpuXzhdVNtLP14XHNFG7gXmDbuBw"
		style="width: 100%; height: 1200px"
		:restriction="{
			latLngBounds: bounds,
			strictBounds: true,
		}"
		:center="center"
		:zoom="2.8"
		:min-zoom="2"
		:max-zoom="8"
		@map-loaded="(map) => { mapInstance.value = map }"
	>
		<Polygon 
			v-for="(countryPolygon, index) in countryPolygons"
			:key="index"
			:options="countryPolygon"
			@click="handleZoneClick(countryPolygon)"
		/>
		<AdvancedMarker
			v-for="(marker, index) in markers"
			:key="index"
			:options="{ position: { lat: marker.lat, lng: marker.lng }, title: marker.title }"
			:pin-options="pinOptions"
			@click="handleMarkerClick(marker)"
		>
			<InfoWindow>
				<div id="content">
					<div id="siteNotice" />
					<h1
						id="firstHeading"
						class="firstHeading"
					>
						05 innovations found!
					</h1>
					<div id="bodyContent">
						<p>No. Innovation Developer: XX</p>
						<p>No. Innovation Implementer: XX</p>
						<p>Location: {{ marker.lat }} {{ marker.lng }}</p>
					</div>
				</div>
			</InfoWindow>
		</AdvancedMarker>
	</GoogleMap>
</template>

<style>
.gm-style-iw-chr {
	display: none;
  visibility: hidden;
}
</style>
